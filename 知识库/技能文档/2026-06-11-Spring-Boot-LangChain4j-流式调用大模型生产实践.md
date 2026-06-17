---
title: "Spring Boot + LangChain4j 流式调用大模型生产实践：从首 Token 延迟到百万级会话架构设计"
source: "Ray的银河技术"
source_url: "https://mp.weixin.qq.com/s/4JMLkfIqIvlW4a6reJcUOQ"
date: "2026-06-11"
tags: [Spring Boot, LangChain4j, 流式调用, 大模型, 架构设计, SSE, WebFlux]
---

# Spring Boot + LangChain4j 流式调用大模型生产实践：从首 Token 延迟到百万级会话架构设计

> 面向对象：有 Spring Boot、微服务、高并发系统经验，希望把 AI 对话、AI 助手、AI Copilot 从"能跑 Demo"升级为"可在线上稳定承载"的工程团队。

很多团队第一次接入大模型时，代码往往是这样的：

```java
String answer = chatModel.chat(userMessage);
return ResponseEntity.ok(answer);
```

在功能验证阶段，这样写没有问题；但一旦进入真实业务，问题会迅速暴露：

1. 用户要等模型完整生成后才能看到结果，首屏感知很差。
2. 一个请求会长时间占住线程，QPS 一上来线程池就被打满。
3. 超长输出、弱网取消、限流、审计、追踪、会话记忆、多模型切换都没有治理点。
4. 当对话服务从单机演进到多实例部署时，本地会话上下文、SSE 长连接和弹性扩缩容之间会互相牵扯。

所以，流式调用的价值从来不只是"打字机效果"，而是把 AI 服务从一次性阻塞 RPC，升级为一条可观测、可治理、可扩展的实时输出链路。

本文不只讲 LangChain4j 的调用方法，而是从协议、线程模型、架构分层、工程治理、生产代码、容量规划几个层面，完整讲清楚：

1. 为什么流式输出是 AI 应用的基础设施能力。
2. Spring Boot 中为什么 WebFlux 比传统 Servlet 更适合此类场景。
3. LangChain4j 的流式回调如何正确桥接为 Reactor 流。
4. 高并发下如何处理限流、背压、取消、超时、熔断、会话记忆和审计。
5. 如何把一套 Demo 升级成可上线的生产级方案。

---

## 一、为什么 AI 对话必须流式化

### 1.1 用户感知的核心不是总耗时，而是 TTFT

在传统同步调用模式下，用户必须等待完整答案生成结束后才能看到响应。假设一次回答总耗时 8 秒，哪怕答案质量很高，用户主观感受依然会认为系统"卡住了"。

而流式模式下，系统可以在 300ms 到 1200ms 内把首个 token 推给前端。这里最关键的指标不是总时长，而是：

1. `TTFT`：Time To First Token，首 token 延迟。
2. `Tokens/s`：流式输出吞吐。
3. `Completion Ratio`：请求发起后最终成功完成的比例。
4. `Cancel Ratio`：用户中途取消比例。

对 AI 聊天产品来说，TTFT 往往比总耗时更决定体验。因为一旦用户看到内容开始输出，就认为系统"已经在工作"。

### 1.2 从资源模型看，流式比阻塞式更适合高并发

传统阻塞式接口的问题不只是慢，而是资源利用方式错误。

当你使用 `chatModel.chat()` 阻塞等待结果时：

1. 业务线程被占住。
2. 网关连接被长时间持有。
3. 下游模型接口在慢速返回时，上游线程只能空等。
4. 当并发上升时，线程数、上下文切换、堆内对象和连接池都会一起膨胀。

这类链路本质上是"外部 I/O 主导型"场景，真正消耗 CPU 的时间远少于等待模型返回 token 的时间。因此，事件驱动 + 非阻塞 I/O 才是更合理的运行模型。

---

## 二、流式大模型调用到底发生了什么

### 2.1 底层并不是"模型一次次回调"，而是 HTTP 分块传输

大部分模型服务商在开启流式输出后，本质都是基于 HTTP chunked transfer 或 SSE 持续返回增量内容。整体链路可以抽象成下面这样：

```text
Browser / App
    |
    |  POST /api/ai/chat/stream
    v
AI Gateway / Chat Service
    |
    |  stream=true
    v
LLM Provider
    |
    |  chunk-1: "你"
    |  chunk-2: "好"
    |  chunk-3: "，下面"
    |  chunk-4: "我来解释"
    v
AI Gateway / Chat Service
    |
    |  SSE / text-event-stream
    v
Browser incremental render
```

也就是说，模型不是等全部文本生成完才返回，而是边解码、边通过网络向上游推送增量结果。后端要做的事情，不是"等待所有结果后统一返回"，而是把这条增量流安全地向前端透传，并在链路两侧补上治理能力。

### 2.2 LangChain4j 在这个过程中扮演什么角色

LangChain4j 的价值不是替代 Spring，而是把模型调用、消息结构、记忆、工具调用等能力抽象成 Java 生态可组合的接口。

在流式场景里，最关键的是它提供的 `StreamingChatModel`。它会把下游模型返回的增量结果，通过回调持续通知业务代码。

这意味着：

1. LangChain4j 解决了"如何和模型流式交互"。
2. Spring WebFlux 解决了"如何把这条流稳定地暴露给客户端"。
3. 你的业务代码要解决"如何治理这条流"。

### 2.3 为什么不建议把流式理解成"边输出边拼字符串"

很多 Demo 的思路是：每来一个 token，就 append 到 `StringBuilder`，然后直接返回给前端。

这只是最表层的实现。线上真正要处理的是：

1. 某个 token 到达很慢，是否算超时。
2. 前端断开连接后，是否继续让模型生成。
3. 流式过程中是否要记录审计日志和成本指标。
4. 长文本输出时如何避免内存无限增长。
5. 多租户配额、模型路由、故障切换如何嵌入流式链路。

所以，流式调用的正确视角不是"字符串流"，而是"受治理的实时事件流"。

---

## 三、生产级架构应该怎么设计

### 3.1 推荐的分层结构

对于 AI 对话、AI 客服、AI 助手一类场景，推荐采用下面这套分层：

```text
接入层
  - Web / App / 小程序 / BFF

输出层
  - SSE / HTTP streaming

会话层
  - Session 管理
  - 用户上下文
  - 消息编排
  - 取消控制

推理层
  - LangChain4j
  - Prompt 组装
  - Tool Calling
  - RAG 检索
  - Multi-model routing

治理层
  - 限流
  - 熔断
  - 重试
  - 降级
  - 审计
  - 成本控制

状态层
  - Redis 会话记忆
  - Kafka 审计事件
  - MySQL/PostgreSQL 对话元数据
  - Metrics / Trace / Log
```

这里最容易被忽略的是治理层与状态层。很多项目直接把 LangChain4j 嵌在 Controller 里，功能能通，但业务一放量就会暴露出治理缺口。

### 3.2 为什么 Spring Boot 场景推荐 WebFlux

如果接口是典型的"CPU 计算型短请求"，Servlet 模型完全够用。但 AI 流式输出不是这种场景，它具有三个特点：

1. 请求持续时间长。
2. 大量时间消耗在等待外部模型返回。
3. 每个请求会产生多次增量输出。

在这种场景下，如果还用阻塞式线程模型，一个请求往往会长期占住一个工作线程，系统容量会被线程数而不是 CPU 算力限制。

WebFlux 的价值在于：

1. 用更少的线程承载更多长连接。
2. 把输出建模成 `Flux`，天然适合表达 token 流、事件流、完成信号和错误信号。
3. 更容易做超时、取消、背压和链路观测。

这里不是说 Servlet 绝对不能做流式，而是从高并发和治理成本看，WebFlux 更顺手，也更接近流式 AI 服务的资源模型。

### 3.3 为什么大多数 AI 输出场景优先选 SSE

如果你的场景是"用户发起一次提问，服务端持续返回结果"，那优先选 SSE。

因为 SSE 有几个非常现实的优点：

1. 基于标准 HTTP，代理层和网关层更容易兼容。
2. 语义清晰，天然就是单向事件推送。
3. 前端接入简单，很多平台都能较低成本支持。
4. 对 AI 输出这类"服务端连续推送"模式足够合适。

只有在以下情况才更建议 WebSocket：

1. 需要客户端随时插入控制命令，例如暂停、继续、切换模式。
2. 需要双向实时协作，例如实时语音、多人协同。
3. 需要在一条连接里承载大量不同类型事件。

大多数文本生成场景中，SSE 的复杂度更低，工程性更强。

---

## 四、从 Demo 到线上，核心链路应该怎样落地

下面给出一套更接近生产的实现骨架。示例以 Spring Boot + WebFlux + LangChain4j 为主，重点不在于逐行 API，而在于系统边界和治理位置。

### 4.1 Maven 依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j</artifactId>
    </dependency>

    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-open-ai</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
    </dependency>

    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-prometheus</artifactId>
    </dependency>
</dependencies>
```

如果你的项目已经采用企业统一 SDK 或私有模型网关，那么 LangChain4j 的 provider 依赖应替换成对应适配器，不要强绑某一家模型厂商。

### 4.2 基础配置

```yaml
server:
  port: 8080

spring:
  application:
    name: ai-stream-chat-service
  data:
    redis:
      host: 127.0.0.1
      port: 6379

management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus

app:
  ai:
    default-model: gpt-4o-mini
    max-history-messages: 12
    max-prompt-chars: 12000
    request-timeout: 40s
    first-token-timeout: 8s
    session-ttl: 12h
    max-output-chars: 32000
```

这里建议你显式配置三类限制：

1. 上下文限制：避免历史消息无限堆积。
2. 时间限制：避免 provider 卡死拖垮线程和连接。
3. 输出限制：避免异常 prompt 触发超长输出占满内存。

### 4.3 请求与事件模型

不要直接把字符串往外推。生产中更推荐输出结构化事件，便于前端、网关和审计系统统一处理。

```java
package com.example.ai.api;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record ChatStreamRequest(
        @NotBlank String sessionId,
        @NotBlank @Size(max = 4000) String message,
        String model,
        String userId,
        Boolean storeHistory
) {
}
```

```java
package com.example.ai.api;

public record ChatStreamEvent(
        String type,
        String sessionId,
        String content,
        Long seq,
        Long timestamp,
        String traceId
) {

    public static ChatStreamEvent token(String sessionId, String content, long seq, String traceId) {
        return new ChatStreamEvent("token", sessionId, content, seq, System.currentTimeMillis(), traceId);
    }

    public static ChatStreamEvent completed(String sessionId, String traceId) {
        return new ChatStreamEvent("completed", sessionId, "", -1L, System.currentTimeMillis(), traceId);
    }

    public static ChatStreamEvent error(String sessionId, String content, String traceId) {
        return new ChatStreamEvent("error", sessionId, content, -1L, System.currentTimeMillis(), traceId);
    }
}
```

这样做有几个好处：

1. 前端可以区分 token、完成、错误、心跳等事件。
2. 中间链路可以直接消费结构化消息。
3. 未来切换到 WebSocket 或消息总线时，事件模型仍然能复用。

### 4.4 LangChain4j 模型装配

```java
package com.example.ai.config;

import dev.langchain4j.model.openai.OpenAiStreamingChatModel;
import dev.langchain4j.model.chat.StreamingChatModel;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.Duration;

@Configuration
public class AiModelConfig {

    @Bean
    public StreamingChatModel streamingChatModel(
            @Value("${llm.base-url}") String baseUrl,
            @Value("${llm.api-key}") String apiKey,
            @Value("${app.ai.default-model}") String modelName,
            @Value("${app.ai.request-timeout}") Duration timeout) {

        return OpenAiStreamingChatModel.builder()
                .baseUrl(baseUrl)
                .apiKey(apiKey)
                .modelName(modelName)
                .timeout(timeout)
                .logRequests(false)
                .logResponses(false)
                .build();
    }
}
```

这里有两个工程建议：

1. 不要在生产环境打开完整请求和响应日志，Prompt、用户隐私和模型输出都可能进入日志系统。
2. 模型路由不要直接写死在 Controller，应单独抽象一层 `ModelRouteService`，后续才能做租户级配额、模型灰度和故障切换。

---

## 五、生产级流式桥接实现

### 5.1 不要在 Controller 里直接写业务逻辑

推荐把流式编排放进独立应用服务中，让 Controller 只负责协议暴露。

```java
package com.example.ai.service;

import com.example.ai.api.ChatStreamEvent;
import com.example.ai.api.ChatStreamRequest;
import reactor.core.publisher.Flux;

public interface AiChatStreamService {

    Flux<ChatStreamEvent> stream(ChatStreamRequest request);
}
```

### 5.2 核心服务实现

下面这段代码的重点不是 API 细节，而是几个关键动作：

1. 预先校验会话、配额和 prompt 大小。
2. 构建历史消息。
3. 通过 `Flux.create` 把 LangChain4j 回调桥接成 Reactor 流。
4. 捕获完成、错误、取消、超时。
5. 记录 TTFT、token 数、时长和最终状态。

```java
package com.example.ai.service.impl;

import com.example.ai.api.ChatStreamEvent;
import com.example.ai.api.ChatStreamRequest;
import com.example.ai.service.AiChatStreamService;
import com.example.ai.service.ChatHistoryService;
import com.example.ai.service.ChatGovernanceService;
import com.example.ai.service.ChatMetricsService;
import dev.langchain4j.data.message.AiMessage;
import dev.langchain4j.data.message.ChatMessage;
import dev.langchain4j.data.message.UserMessage;
import dev.langchain4j.model.chat.StreamingChatModel;
import dev.langchain4j.model.chat.response.ChatResponse;
import dev.langchain4j.model.chat.response.StreamingChatResponseHandler;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.slf4j.MDC;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Flux;

import java.time.Duration;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeoutException;
import java.util.concurrent.atomic.AtomicBoolean;
import java.util.concurrent.atomic.AtomicLong;

@Slf4j
@Service
@RequiredArgsConstructor
public class DefaultAiChatStreamService implements AiChatStreamService {

    private final StreamingChatModel streamingChatModel;
    private final ChatHistoryService chatHistoryService;
    private final ChatGovernanceService chatGovernanceService;
    private final ChatMetricsService chatMetricsService;

    @Value("${app.ai.first-token-timeout}")
    private Duration firstTokenTimeout;

    @Value("${app.ai.request-timeout}")
    private Duration requestTimeout;

    @Value("${app.ai.max-output-chars}")
    private int maxOutputChars;

    @Override
    public Flux<ChatStreamEvent> stream(ChatStreamRequest request) {
        chatGovernanceService.validateRequest(request);
        chatGovernanceService.acquireQuota(request);

        final String traceId = MDC.get("traceId");
        final long start = System.nanoTime();
        final AtomicLong seq = new AtomicLong(0);
        final AtomicBoolean firstTokenArrived = new AtomicBoolean(false);

        return Flux.<ChatStreamEvent>create(sink -> {
            List<ChatMessage> messages = new ArrayList<>(chatHistoryService.loadHistory(request.sessionId()));
            messages.add(UserMessage.from(request.message()));

            StringBuilder output = new StringBuilder(1024);

            streamingChatModel.chat(messages, new StreamingChatResponseHandler() {
                @Override
                public void onPartialResponse(String partialResponse) {
                    if (partialResponse == null || partialResponse.isEmpty()) {
                        return;
                    }
                    if (output.length() + partialResponse.length() > maxOutputChars) {
                        sink.error(new IllegalStateException("model output exceeded max-output-chars"));
                        return;
                    }
                    output.append(partialResponse);
                    long currentSeq = seq.incrementAndGet();
                    if (firstTokenArrived.compareAndSet(false, true)) {
                        chatMetricsService.recordFirstTokenLatency(request, Duration.ofNanos(System.nanoTime() - start));
                    }
                    sink.next(ChatStreamEvent.token(request.sessionId(), partialResponse, currentSeq, traceId));
                }

                @Override
                public void onCompleteResponse(ChatResponse completeResponse) {
                    AiMessage aiMessage = completeResponse.aiMessage();
                    if (Boolean.TRUE.equals(request.storeHistory())) {
                        chatHistoryService.appendRound(request.sessionId(), request.message(), aiMessage.text());
                    }
                    chatMetricsService.recordCompleted(request, output.toString(), Duration.ofNanos(System.nanoTime() - start));
                    sink.next(ChatStreamEvent.completed(request.sessionId(), traceId));
                    sink.complete();
                }

                @Override
                public void onError(Throwable error) {
                    chatMetricsService.recordFailed(request, error, Duration.ofNanos(System.nanoTime() - start));
                    sink.next(ChatStreamEvent.error(request.sessionId(), error.getMessage(), traceId));
                    sink.error(error);
                }
            });
        })
        .timeout(firstTokenTimeout, Flux.error(new TimeoutException("first token timeout")))
        .timeout(requestTimeout)
        .doOnCancel(() -> {
            chatMetricsService.recordCancelled(request, Duration.ofNanos(System.nanoTime() - start));
            chatGovernanceService.onClientCancel(request);
            log.info("chat stream cancelled, sessionId={}", request.sessionId());
        })
        .doFinally(signalType -> chatGovernanceService.releaseQuota(request))
        .onErrorMap(TimeoutException.class, ex -> new IllegalStateException("chat stream timeout", ex));
    }
}
```

### 5.3 这段实现里最关键的几个工程点

#### 第一，首 token 超时和总超时要分开

如果 8 秒都没收到首 token，用户通常已经认为系统失败；但一旦开始输出，总时长可以比首 token 超时更宽松。因此，`first-token-timeout` 和 `request-timeout` 应分层控制。

#### 第二，不要在 `onPartialResponse` 里做阻塞 I/O

很多人会在每个 token 回调里：

1. 写数据库。
2. 调 Redis。
3. 发同步审计请求。
4. 做复杂字符串处理。

这会直接拖慢整个流式输出链路。正确方式是：

1. 增量事件只做轻量投递。
2. 审计、埋点、落盘走异步队列。
3. 真正的持久化放在完成阶段，或做批量缓冲后异步落地。

#### 第三，取消一定要被感知

用户关闭页面或切换会话时，如果服务端还让模型继续生成，就是白白烧钱。线上系统必须把"客户端取消"当作正式状态处理，并尽可能中断下游生成任务或至少停止继续透传。

---

## 六、Controller 层如何优雅暴露 SSE

```java
package com.example.ai.controller;

import com.example.ai.api.ChatStreamEvent;
import com.example.ai.api.ChatStreamRequest;
import com.example.ai.service.AiChatStreamService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.MediaType;
import org.springframework.http.codec.ServerSentEvent;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Flux;

@RestController
@RequestMapping("/api/ai/chat")
@RequiredArgsConstructor
public class AiChatStreamController {

    private final AiChatStreamService aiChatStreamService;

    @PostMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<ServerSentEvent<ChatStreamEvent>> stream(@Valid @RequestBody ChatStreamRequest request) {
        return aiChatStreamService.stream(request)
                .map(event -> ServerSentEvent.<ChatStreamEvent>builder()
                        .event(event.type())
                        .id(event.seq() == null ? null : String.valueOf(event.seq()))
                        .data(event)
                        .build());
    }
}
```

这里使用 `POST` 而不是 `GET`，是因为生产环境中请求体通常包含：

1. 用户问题。
2. 会话 ID。
3. 模型参数。
4. 业务侧扩展字段。

这些内容不适合塞进 query string。前端可以使用 `fetch` + streaming reader 处理，而不是执着于 `EventSource` 只能发 GET 的限制。

---

## 七、会话记忆怎样设计才适合多实例部署

### 7.1 本地内存记忆为什么只能用于单机 Demo

如果你把聊天历史保存在本地 `Map` 或 JVM 内存里，单机调试时非常方便，但多实例部署后会立刻出现三个问题：

1. 请求打到不同实例时，上下文丢失。
2. 实例重启后会话全部消失。
3. 无法做统一 TTL、容量管理和跨实例审计。

所以，线上必须把会话状态外置。

### 7.2 推荐的状态拆分

不要把所有状态都塞进一个地方。更合理的做法是：

1. Redis：短期会话上下文、最近 N 轮对话、会话 TTL。
2. MySQL/PostgreSQL：会话元数据、工单关联、用户维度配置。
3. Kafka：token 流审计、完成事件、异常事件、成本事件。
4. 对象存储：超长原文、附件、引用文档。

### 7.3 Redis 会话历史实现示例

```java
package com.example.ai.service.impl;

import com.example.ai.service.ChatHistoryService;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import dev.langchain4j.data.message.AiMessage;
import dev.langchain4j.data.message.ChatMessage;
import dev.langchain4j.data.message.UserMessage;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.redis.core.ReactiveStringRedisTemplate;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Mono;

import java.time.Duration;
import java.util.ArrayList;
import java.util.List;

@Service
@RequiredArgsConstructor
public class RedisChatHistoryService implements ChatHistoryService {

    private static final String KEY_PREFIX = "ai:chat:history:";

    private final ReactiveStringRedisTemplate redisTemplate;
    private final ObjectMapper objectMapper;

    @Value("${app.ai.max-history-messages}")
    private int maxHistoryMessages;

    @Value("${app.ai.session-ttl}")
    private Duration sessionTtl;

    @Override
    public List<ChatMessage> loadHistory(String sessionId) {
        String json = redisTemplate.opsForValue().get(KEY_PREFIX + sessionId).block();
        if (json == null || json.isBlank()) {
            return new ArrayList<>();
        }
        try {
            return objectMapper.readValue(json, new TypeReference<List<SimpleChatRecord>>() {})
                    .stream()
                    .flatMap(record -> List.of(
                            UserMessage.from(record.user()),
                            AiMessage.from(record.assistant())
                    ).stream())
                    .toList();
        } catch (Exception e) {
            return new ArrayList<>();
        }
    }

    @Override
    public void appendRound(String sessionId, String userMessage, String assistantMessage) {
        List<SimpleChatRecord> records = new ArrayList<>(loadSimple(sessionId));
        records.add(new SimpleChatRecord(userMessage, assistantMessage));
        if (records.size() > maxHistoryMessages) {
            records = records.subList(records.size() - maxHistoryMessages, records.size());
        }
        try {
            String json = objectMapper.writeValueAsString(records);
            redisTemplate.opsForValue().set(KEY_PREFIX + sessionId, json, sessionTtl).block();
        } catch (Exception ignored) {
        }
    }

    private List<SimpleChatRecord> loadSimple(String sessionId) {
        String json = redisTemplate.opsForValue().get(KEY_PREFIX + sessionId).block();
        if (json == null || json.isBlank()) {
            return new ArrayList<>();
        }
        try {
            return objectMapper.readValue(json, new TypeReference<List<SimpleChatRecord>>() {});
        } catch (Exception e) {
            return new ArrayList<>();
        }
    }

    private record SimpleChatRecord(String user, String assistant) {
    }
}
```

上面为了突出思路，使用了相对直观的实现。真正线上版本还应继续完善：

1. 不要在反应式链路里随意 `block()`，最好统一边界。
2. 增加消息裁剪与敏感字段脱敏。
3. 增加会话版本号，避免并发写覆盖。
4. 对异常反序列化做监控，而不是静默吞掉。

---

## 八、高并发下真正会踩的坑

### 8.1 背压不是可选项

流式输出并不意味着链路天然安全。一个常见问题是：

1. 下游模型生成速度很快。
2. 前端网络较慢或消费较慢。
3. 服务端持续积压未发送数据。
4. 最终触发堆内存增长或连接写阻塞。

所以，系统必须有背压意识。常见手段包括：

1. 对单连接缓冲区做上限控制。
2. 对单次输出总字符数做限制。
3. 对超慢消费者主动断流。
4. 对 token 级审计改成异步批量落地。

不要把"一个 token 一个事件"机械化理解成绝对正确。在高吞吐场景里，按固定字符数或按 20ms 到 50ms 小窗口做聚合发送，往往更划算。

### 8.2 限流要按租户、模型、会话多维治理

AI 服务的限流不能只看接口 QPS。因为两个请求的成本差异可能非常大。

更合理的治理维度包括：

1. 每租户并发会话数。
2. 每用户每分钟请求次数。
3. 每模型每分钟 token 预算。
4. 单会话输出长度上限。
5. 工具调用次数上限。

推荐把这些规则收敛到统一治理服务，而不是散落在 Controller、Filter 和业务逻辑里。

```java
package com.example.ai.service;

import com.example.ai.api.ChatStreamRequest;

public interface ChatGovernanceService {

    void validateRequest(ChatStreamRequest request);

    void acquireQuota(ChatStreamRequest request);

    void releaseQuota(ChatStreamRequest request);

    void onClientCancel(ChatStreamRequest request);
}
```

这种设计的好处是，后续接入 Redis 限流、Sentinel、Resilience4j 甚至企业网关配额体系时，不需要大改主流程。

### 8.3 不能把全部历史消息无脑塞给模型

这是 AI 应用里非常高频的隐性性能问题。

很多人为了"让模型记住上下文"，会把全量聊天记录不断追加进去。结果是：

1. Prompt 越来越大。
2. 成本不断上涨。
3. TTFT 越来越慢。
4. 长会话极易触发 token 上限或 provider 拒绝。

正确方式应该是分层记忆：

1. 工作记忆：最近几轮消息，直接参与当前推理。
2. 摘要记忆：对更久的历史进行总结后再注入。
3. 事实记忆：用户画像、偏好、账号状态等结构化信息。
4. 外部知识：通过 RAG 检索按需注入，而不是永久堆在 prompt 中。

### 8.4 取消、重试、幂等必须一起考虑

在真实流式场景里，一个请求可能出现：

1. 用户发起后立即取消。
2. 网关超时但下游模型仍在继续生成。
3. 客户端重连后重复请求。
4. 上游重复提交同一个 messageId。

因此建议把每轮请求都定义唯一的 `requestId`，并做如下处理：

1. 已完成请求不可重复入账。
2. 取消请求标记状态，避免后续重复写历史。
3. 审计与落库使用幂等键。
4. 计费与配额核销使用最终状态统一结算。

---

## 九、可观测性怎么做，才不是"只看接口 RT"

AI 流式服务最忌讳只看传统接口耗时。因为传统 RT 无法反映用户真实体验。

推荐至少监控以下指标：

1. `ai_chat_ttft_ms`：首 token 延迟。
2. `ai_chat_duration_ms`：总会话耗时。
3. `ai_chat_output_chars`：输出长度。
4. `ai_chat_cancel_total`：取消次数。
5. `ai_chat_error_total`：错误次数。
6. `ai_chat_provider_timeout_total`：模型侧超时次数。
7. `ai_chat_inflight_sessions`：当前活跃会话数。
8. `ai_chat_tokens_total`：输入输出 token 消耗。

示例：

```java
package com.example.ai.service.impl;

import com.example.ai.api.ChatStreamRequest;
import com.example.ai.service.ChatMetricsService;
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import org.springframework.stereotype.Service;

import java.time.Duration;

@Service
public class MicrometerChatMetricsService implements ChatMetricsService {

    private final MeterRegistry registry;

    public MicrometerChatMetricsService(MeterRegistry registry) {
        this.registry = registry;
    }

    @Override
    public void recordFirstTokenLatency(ChatStreamRequest request, Duration duration) {
        Timer.builder("ai_chat_ttft_ms")
                .tag("model", safeModel(request.model()))
                .register(registry)
                .record(duration);
    }

    @Override
    public void recordCompleted(ChatStreamRequest request, String output, Duration duration) {
        Timer.builder("ai_chat_duration_ms")
                .tag("model", safeModel(request.model()))
                .register(registry)
                .record(duration);
        Counter.builder("ai_chat_completed_total")
                .tag("model", safeModel(request.model()))
                .register(registry)
                .increment();
    }

    @Override
    public void recordFailed(ChatStreamRequest request, Throwable error, Duration duration) {
        Counter.builder("ai_chat_error_total")
                .tag("model", safeModel(request.model()))
                .tag("error", error.getClass().getSimpleName())
                .register(registry)
                .increment();
    }

    @Override
    public void recordCancelled(ChatStreamRequest request, Duration duration) {
        Counter.builder("ai_chat_cancel_total")
                .tag("model", safeModel(request.model()))
                .register(registry)
                .increment();
    }

    private String safeModel(String model) {
        return model == null || model.isBlank() ? "default" : model;
    }
}
```

真正成熟的系统还应该把 trace 打通，让一条请求能够串起：

1. 网关请求。
2. 会话装配。
3. 检索调用。
4. 工具调用。
5. 模型推理。
6. SSE 输出。
7. 异步审计落地。

这样你才能真正定位"慢"到底慢在检索、模型、网络还是前端消费。

---

## 十、故障治理：超时、熔断、降级不能缺

### 10.1 AI 服务必须承认下游模型不稳定

再强的大模型服务商，也会有抖动、超时、限流和突发失败。线上系统必须内建以下策略：

1. 连接超时。
2. 首 token 超时。
3. 总时长超时。
4. 熔断。
5. 重试。
6. 降级。

但要注意，流式请求不是所有错误都适合自动重试。因为一旦已经输出部分 token，再自动重试通常会导致语义重复和客户端混乱。

所以更合理的策略是：

1. 未收到首 token 前失败：允许有限重试。
2. 已经开始输出后失败：直接结束并返回明确错误事件。
3. Provider 整体异常：切换备用模型或返回降级答案。

### 10.2 降级不是"报错"，而是有层次地退化服务

可落地的降级路径通常包括：

1. 从高阶模型切换到便宜模型。
2. 关闭 RAG、只保留基础问答。
3. 关闭复杂工具调用、只保留纯文本生成。
4. 缩短历史上下文窗口。
5. 在极端高峰时返回异步任务模式，而不是强行实时流式。

很多系统失败，不是因为"没接模型"，而是因为没有设计降级层级。

---

## 十一、前端如何消费流式结果

如果前端使用浏览器环境，推荐用 `fetch` 的流式读取方式消费 `POST` SSE。

```javascript
async function streamChat(payload, onEvent) {
  const response = await fetch("/api/ai/chat/stream", {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify(payload)
  });

  const reader = response.body.getReader();
  const decoder = new TextDecoder("utf-8");
  let buffer = "";

  while (true) {
    const { value, done } = await reader.read();
    if (done) break;
    buffer += decoder.decode(value, { stream: true });

    const chunks = buffer.split("\n\n");
    buffer = chunks.pop() || "";

    for (const chunk of chunks) {
      const line = chunk.split("\n").find(item => item.startsWith("data:"));
      if (!line) continue;
      const data = JSON.parse(line.slice(5).trim());
      onEvent(data);
    }
  }
}
```

前端也要有治理意识，而不只是拼字符串：

1. `token` 事件用于增量渲染。
2. `completed` 事件用于结束状态确认。
3. `error` 事件用于提示与重试。
4. 页面切换或用户停止时，应主动取消请求。

---

## 十二、一个更真实的业务案例

假设你做的是在线教育平台的 AI 答疑助手，业务目标如下：

1. 日均 500 万次问答请求。
2. 高峰每秒 6000 到 10000 次会话创建。
3. 80% 的问题只需要纯文本回答。
4. 15% 的问题需要检索题库与知识点。
5. 5% 的问题需要调用工具，例如读取错题本或学习记录。

这时一个合理的线上架构是：

1. 接入层由网关承接鉴权与租户路由。
2. Chat Service 负责 SSE 输出与会话治理。
3. RAG Service 负责检索和片段裁剪。
4. Tool Service 负责工具调用隔离。
5. Redis 保存短会话状态。
6. Kafka 异步落地审计、指标和成本事件。
7. Prometheus + Grafana 监控 TTFT、错误率、活跃流数和 token 消耗。

在高峰期，系统最优先保护的不是"每次都返回最强模型结果"，而是：

1. 首 token 尽快出来。
2. 服务不雪崩。
3. 配额不被少数大请求耗尽。
4. 异常时可以有序降级。

这就是 AI 服务和传统 CRUD 服务最大的不同：它不是纯事务型系统，而是"实时交互 + 概率推理 + 成本敏感"的复合系统。

---

## 十三、部署与网关层注意事项

### 13.1 反向代理必须正确支持流式透传

如果你的服务前面有 Nginx、Ingress 或 API Gateway，需要重点确认：

1. 不要缓冲整个响应再返回。
2. 连接超时时间要覆盖长会话。
3. 上游空闲超时不能过短。
4. 负载均衡策略要考虑长连接特性。

Nginx 常见配置示例：

```nginx
location /api/ai/chat/stream {
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_buffering off;
    proxy_cache off;
    chunked_transfer_encoding on;
    proxy_read_timeout 300s;
    proxy_send_timeout 300s;
    proxy_pass http://ai-chat-service;
}
```

如果这里没配置好，你在应用层写得再漂亮，也可能被网关缓冲成"假流式"。

### 13.2 Kubernetes 扩容不能只盯 CPU

AI 流式服务扩容更应该关注：

1. 活跃会话数。
2. 下游 provider 调用并发。
3. TTFT 抖动。
4. JVM 堆占用。
5. 网络连接数。

很多时候，CPU 还没高，系统已经因为长连接、缓冲区和下游限流进入退化状态。

因此更合理的 HPA 指标通常是：

1. `inflight sessions`
2. `provider pending requests`
3. `p95 TTFT`
4. `memory working set`

---

## 十四、代码层面还应补哪些生产能力

如果你准备把本文方案真正用于线上，建议继续补齐以下模块：

1. `ModelRouteService`：按租户、地区、成本、稳定性路由模型。
2. `PromptPolicyService`：统一处理系统提示词、敏感词和上下文裁剪。
3. `ChatAuditEventPublisher`：把完成、失败、取消、计费事件异步发往 Kafka。
4. `ConversationSummaryService`：长会话自动摘要，避免上下文膨胀。
5. `ToolExecutionGuard`：限制工具调用次数、超时和权限范围。
6. `TenantQuotaService`：按租户做并发、token、预算控制。

这些能力决定了你的系统是一个"流式对话接口"，还是一个"可治理的 AI 平台能力"。

---

## 十五、最容易犯错的几个认知误区

### 误区一：流式只是前端体验优化

错。流式同时改变了后端的线程模型、网关行为、配额结算、日志采集和故障治理方式。

### 误区二：只要用了 LangChain4j 就算完成 AI 工程化

错。LangChain4j 只是模型交互与编排层，工程化重点仍然是状态、治理和可观测性。

### 误区三：把全量历史上下文都传给模型更智能

错。无限堆上下文只会拖慢 TTFT、推高成本，并增加失败概率。

### 误区四：SSE 输出了就天然高并发

错。没有背压、限流、取消和缓冲治理，流式链路同样会被打爆。

### 误区五：接口 RT 正常就说明体验正常

错。AI 交互更关键的是 TTFT、流速、取消率和最终完成率。

---

## 十六、结语：真正的生产升级，不是"能流式输出"，而是"能稳定承载流式输出"

Spring Boot + LangChain4j 做流式大模型调用，入门并不难，难的是把它做成线上能力。

从架构角度看，这件事至少包含三层：

1. 协议层：把下游模型增量输出稳定转成上游 SSE/Streaming Response。
2. 运行层：基于 WebFlux 构建适合长连接和外部 I/O 的线程模型。
3. 治理层：补齐限流、取消、超时、熔断、记忆、审计、观测和成本控制。

一个真正成熟的 AI 流式系统，目标从来不是"演示打字机效果"，而是：

1. 首 token 快。
2. 链路可控。
3. 成本可管。
4. 故障可降级。
5. 架构可横向扩展。

当你用这个视角再回看流式 LLM，就会发现它本质上不是一个简单接口问题，而是一套新的实时推理基础设施问题。

这，才是 Spring Boot + LangChain4j 流式调用大模型在生产环境中的真正价值。
