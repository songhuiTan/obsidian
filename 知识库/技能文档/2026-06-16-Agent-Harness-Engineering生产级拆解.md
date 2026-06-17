---
title: "从 REPL 到生产系统：AI Agent Harness Engineering 深度拆解与工程落地"
source: "银河技术 / Ray的银河技术"
source_url: "https://mp.weixin.qq.com/s/fG-jjoZv3gHmAAY8JkQm_g"
date: "2026-06-16"
tags: [AI, Agent, Harness, 工程化, 生产系统, 架构, 分布式]
---

# 从 REPL 到生产系统：AI Agent Harness Engineering 深度拆解与工程落地

> 本文系统回答一个难题：如何把一个在实验环境里能工作的 Agent，升级为一套可服务真实业务、可承受高并发、可持续演进的 Agent Harness Runtime。

![封面](../assets/2026-06-16-Agent-Harness-Engineering/img_001.jpg)

---

## 一、为什么很多 Agent Demo 一上生产就失控

真实业务中暴露的典型问题：

- 同一请求有时 2 秒返回，有时 20 秒超时
- 模型偶发调用不存在的 Tool，或参数格式漂移
- Tool 重试产生重复扣款、重复派单、重复写库
- 上下文越积越长，Token 成本线性飙升
- 会话执行到一半容器被驱逐，整个任务无法恢复
- 模型供应商抖动时全链路雪崩
- 没有可回放的决策轨迹，排障只能靠猜

**关键结论：** Agent 的难点从来不只是 Prompt 或模型能力，而是围绕模型构建出的那一整套运行时基础设施。

---

## 二、Harness Engineering 到底是什么

| 概念 | 解决什么问题 | 关注点 |
|------|-------------|--------|
| **Framework** | "怎么写" | 组织 Prompt、注册 Tool、多步推理、Memory |
| **Harness** | "怎么跑" | 请求路由、Tool 鉴权/限流/幂等、会话状态持久化、checkpoint、灰度熔断降级、可观测性、成本治理 |

**定位：** 位于 LLM 与业务系统之间的一层**工程控制面 + 执行面 + 治理面**。如果说 Framework 更像 SDK，Harness 更像一套 **Agent 操作系统**。

---

## 三、系统视角：Agent 是"带状态的分布式事务协调者"

生产级 Agent 更接近下面这类系统：

- 一个可中断、可恢复的**状态机**
- 一个会进行外部 I/O 的**编排器**
- 一个连接多个业务域服务的**决策协调器**
- 一个对结果质量、执行成本和风险敞口负责的**运行时**

设计 Agent Harness 时必须主动套用成熟的分布式系统方法论：

| 不应理解为 | 应理解为 |
|-----------|---------|
| 更智能的函数调用器 | LLM 驱动的工作流执行引擎 |
| 聊天机器人 | 带状态的分布式事务协调者 |
| Prompt 工程 | 运行时基础设施工程 |

---

## 四、生产级 Agent Harness 的五层架构

```
┌────────────────────────────────────────────────────────┐
│                   API / Event Ingress                   │
├────────────────────────────────────────────────────────┤
│               Agent Orchestration Runtime               │
│  Step Engine / Planner / State Machine / Budget Control │
├────────────────────────────────────────────────────────┤
│      LLM Gateway            │      Tool Execution Bus    │
│  Routing / Fallback / QoS   │  Validation / Timeout / ACL│
├────────────────────────────────────────────────────────┤
│  Session Store │ Checkpoint │ Memory │ Policy │ Audit   │
├────────────────────────────────────────────────────────┤
│          PostgreSQL / Redis / Kafka / Object Storage    │
└────────────────────────────────────────────────────────┘
```

| 层 | 职责 |
|----|------|
| **接入层** | HTTP/gRPC/WebSocket/MQ，API 鉴权，配额与优先级 |
| **编排层** | 执行上下文构建，驱动 ReAct/Plan-Execute/DAG，控制步数/重试/超时 |
| **模型接入层** | 多模型路由，熔断限流降级，Token 计费，Prompt 注入 |
| **Tool 执行层** | 统一动作总线：注册、校验、超时、幂等、权限、审批 |
| **状态与治理层** | Session Store、Checkpoint、Trace/Log/Metrics、审计、策略、成本、安全 |

---

## 五、五大核心设计原则

### 5.1 LLM 必须被当成不稳定依赖

模型可能出现：超时、输出不符合 schema、幻觉 Tool 名称、长上下文退化、供应商抖动、配额耗尽。

**必须内建：** 请求超时、重试策略、供应商降级、模型切换、响应校验、预算熔断。

### 5.2 Tool Call 本质就是 RPC

流程：模型输出动作意图 → 运行时参数校验 → 本地调用/HTTP/gRPC/MQ 执行 → 结果回灌。

**必须具备：** 契约管理、超时隔离、幂等控制、重试策略、熔断限流、结果摘要。

### 5.3 状态必须可持久化、可回放、可恢复

只要涉及多步推理、多次 Tool 调用、多次写操作，就必须有：执行上下文持久化、步骤级 checkpoint、中断恢复、轨迹回放。

### 5.4 副作用操作必须显式治理

扣费、发券、派单、改库存等必须经过：风险分级、权限校验、审批确认、幂等键、补偿机制。

### 5.5 可观测性不是附加项，而是主流程的一部分

必须能回答：为什么这个请求耗时 8 秒？为什么选了 Tool A 而不是 B？为什么第三步失败没有重试？

---

## 六、核心数据模型设计

### AgentRequest

```java
public record AgentRequest(
    String requestId, String tenantId, String sessionId,
    String userId, String channel, String userInput,
    Map<String, Object> context, ExecutionPolicy policy
) {}
```

### ExecutionPolicy（每次请求的"运行配额单"）

```java
public record ExecutionPolicy(
    int maxSteps, int maxModelRetries, int maxToolRetries,
    long totalTimeoutMs, long singleToolTimeoutMs,
    int maxInputTokens, int maxOutputTokens,
    boolean allowParallelTools,
    boolean requireHumanApprovalForCriticalOps,
    String fallbackMode
) {}
```

### AgentStep

```java
public record AgentStep(
    int index, StepType type, String plannerThought,
    ToolCall toolCall, ToolResult toolResult,
    StepStatus status, long startAt, long endAt, String traceId
) {}
```

### ToolCall 与 ToolResult

```java
public record ToolCall(
    String toolName, Map<String, Object> arguments,
    String idempotencyKey, String riskLevel, long timeoutMs
) {}

public record ToolResult(
    String toolName, String executionId, ToolStatus status,
    String summary,          // 回灌给模型的结果摘要
    String rawPayloadRef,    // 原始结果在对象存储中的引用
    boolean retryable, long durationMs
) {}
```

两个生产级关键点：**summary** 避免大对象塞回上下文；**rawPayloadRef** 防止上下文污染。

### SessionSnapshot

```java
public record SessionSnapshot(
    String sessionId, int currentStep,
    List<Message> compactContext, List<AgentStep> finishedSteps,
    String checkpointState, String status
) {}
```

---

## 七、编排引擎设计（状态机化）

**反模式：** `while(!done) { llm.call → parse → tool.execute → messages.add }` — 能演示但无法治理。

**推荐状态枚举：**

```java
public enum RuntimeState {
    CREATED, LOADING_CONTEXT, PLANNING, VALIDATING_ACTION,
    WAITING_APPROVAL, EXECUTING_TOOL, SUMMARIZING_RESULT,
    CHECKPOINTING, COMPLETED, DEGRADED, FAILED, CANCELLED
}
```

**Orchestrator 核心骨架（简化版）：**

```java
public AgentResponse execute(AgentRequest request) {
    RuntimeContext ctx = RuntimeContext.initialize(request);
    long deadline = System.currentTimeMillis() + request.policy().totalTimeoutMs();
    transition(ctx, RuntimeState.LOADING_CONTEXT);
    restoreIfNeeded(ctx);

    while (!ctx.isTerminal()) {
        ensureDeadline(deadline);
        transition(ctx, RuntimeState.PLANNING);
        PlannerDecision decision = llmGateway.plan(ctx.toPlannerInput());
        if (decision.isFinalAnswer()) { ctx.complete(); break; }

        transition(ctx, RuntimeState.VALIDATING_ACTION);
        ValidatedAction action = policyEngine.validate(decision.toolCall(), ctx);
        if (action.requiresApproval()) {
            transition(ctx, RuntimeState.WAITING_APPROVAL);
            if (!waitApproval(action, ctx)) { ctx.degrade(); break; }
        }

        transition(ctx, RuntimeState.EXECUTING_TOOL);
        ToolResult result = toolExecutionBus.execute(action, ctx);
        ctx.appendObservation(result.summary());

        transition(ctx, RuntimeState.CHECKPOINTING);
        persistCheckpoint(ctx);
        if (ctx.stepCountExceeded()) ctx.degrade("max steps exceeded");
    }
    traceRecorder.finish(ctx);
    return ctx.toResponse();
}
```

**状态机化的价值：** 步骤级重试、中断恢复、步骤级 SLA 统计、人工审批挂起、分布式 Worker 接力、失败归因。

---

## 八、LLM Gateway

### Gateway 的职责不是"转发请求"，而是"治理模型"

- 供应商抽象、模型路由、超时与重试、降级链
- 令牌桶限流、Token 统计、成本归集
- Prompt 注入系统变量、输出 schema 校验

### 多模型路由策略

1. 简单分类任务走小模型
2. 高风险决策走高质量模型
3. 长上下文走大窗口模型
4. 高峰期自动切到成本更低的备份模型
5. 供应商错误率连续升高时触发熔断

### Gateway 实现骨架

```java
public PlannerDecision plan(PlannerInput input) {
    List<String> routeChain = providerRouter.route(input);
    for (String providerName : routeChain) {
        CircuitBreaker breaker = breakerRegistry.get(providerName);
        if (!breaker.allowRequest()) continue;
        try {
            LlmProvider provider = providers.get(providerName);
            LlmResponse response = provider.invoke(input);
            responseGuard.validateOrThrow(response, input.expectedSchema());
            costMeter.record(providerName, response.usage());
            breaker.recordSuccess();
            return PlannerDecision.from(response);
        } catch (Exception ex) {
            breaker.recordFailure(ex);
        }
    }
    throw new LlmGatewayException("no available llm provider");
}
```

---

## 九、Tool Execution Bus

### Tool 元数据定义

每个 Tool 至少注册：参数 schema、结果 schema、风险等级、是否可重试、是否幂等、默认超时、所属业务域、所需权限、限流策略、审批策略。

```java
public record ToolDescriptor(
    String name, String description, JsonSchema inputSchema,
    JsonSchema outputSchema, boolean idempotent,
    boolean retryable, String riskLevel, long timeoutMs,
    Set<String> requiredPermissions, boolean approvalRequired
) {}
```

### 结果摘要（关键设计）

大对象（订单明细、风控报告、搜索结果集）不应原样塞回模型上下文：
1. 原始结果落库存证据（rawPayloadRef）
2. 生成结构化摘要（summary）
3. 只把摘要回灌给模型

好处：降低 Token 成本、减少上下文污染、防止敏感数据暴露。

---

## 十、记忆与上下文工程

### 生产级上下文的四类来源

| 来源 | 内容 |
|------|------|
| 系统上下文 | 平台规则、行为边界、输出要求 |
| 会话上下文 | 当前用户本轮与历史交互摘要 |
| 业务上下文 | 订单、用户画像、工单、权限、资源状态 |
| 外部知识上下文 | 检索结果、策略文档、FAQ、知识库片段 |

### 三段式装配模型

```
System Policy Block + Task Objective Block + Selected Evidence Block
```

### 三个上下文治理原则

1. 只给与当前任务强相关的证据，不给"可能有用"的材料
2. 只给摘要化的状态，不给未经裁剪的原始对象
3. 只保留可解释的历史，不保留噪声对话

---

## 十一、高并发设计

### 四层隔离

| 层 | 策略 |
|----|------|
| 入口隔离 | 不同租户/业务/优先级流量分开 |
| 模型隔离 | 不同模型池独立限流 |
| Tool 隔离 | 高风险/慢/外部依赖 Tool 分仓治理 |
| 存储隔离 | 热状态/冷轨迹/审计明细分库存放 |

### 同步与异步分层

| 类型 | SLA | 示例 |
|------|-----|------|
| 实时交互型 | 秒级 | 客服问答、助手建议 |
| 近实时执行型 | 几秒~几十秒 | 复杂审批编排 |
| 后台批处理型 | 分钟级 | 批量工单处理 |

### 并行 Tool 执行（依赖图驱动）

```java
public List<List<ToolCall>> groupExecutableBatches(List<ToolCallNode> dag) {
    // 基于依赖图分批：无依赖的同批并行，有依赖的等上游完成
}
```

### 背压设计

API 入口令牌桶、模型池并发上限、Tool 级舱壁隔离、MQ 消费滞后监控、会话级预算熔断。

---

## 十二、幂等、补偿与一致性

### 必须做幂等的动作

创建退款单、发券、提交审批、调拨库存、发送通知、外部系统回调。

### 分层一致性策略

| 层 | 策略 |
|----|------|
| 会话层 | 允许短暂不一致，但要可恢复 |
| 业务核心层 | 保证关键状态正确性 |
| 观测审计层 | 允许最终一致，但不能丢 |

---

## 十三、安全与治理

### 三层保险

1. **输入清洗** + 模式识别（防 Prompt Injection）
2. **Tool 访问策略**硬限制
3. **输出动作二次校验**

### Tool 三层授权

- 租户级授权（某租户是否具备某类 Tool 能力）
- 用户级授权（当前用户是否能发起该类操作）
- 场景级授权（当前会话上下文是否满足业务前置条件）

### 审批闸门

高风险动作引入审批态：金额超阈值退款、批量修改资源、删除知识库内容、发起外部通知、写生产配置。

---

## 十四、可观测性

### Trace 拆分建议

```
agent.ingress → agent.context.assemble → agent.llm.plan
→ agent.tool.validate → agent.tool.execute
→ agent.result.summarize → agent.checkpoint.persist
→ agent.response.finalize
```

### 关键 Metrics

请求成功率、P50/P95/P99 延迟、平均步数、平均每步 Token 消耗、Tool 超时率、模型降级率、人工接管率、单租户成本、单 Tool 错误率。

### 审计日志 vs 业务日志分开

审计日志关注"谁发起了什么请求、模型决定了什么动作、动作是否被执行、影响了哪些资源"——结构化 DecisionTrace 对象用于排障。

---

## 十五、成本控制

### 五个核心抓手

| 抓手 | 做法 |
|------|------|
| 输入预算 | 限制上下文注入规模 |
| 步骤预算 | 限制最大推理轮数 |
| 模型预算 | 简单任务优先小模型 |
| 结果摘要 | 大结果不原样回灌 |
| 缓存与复用 | 相似任务复用历史结论 |

---

## 十六、从单机到平台的演进路线

### Phase 1：单服务验证期

单体服务 + 单模型 + 本地 Tool + Redis session + PostgreSQL 轨迹 → 跑通最小闭环。

### Phase 2：服务化治理期

LLM Gateway 独立 + Tool Registry 独立 + MQ 异步任务 + 审计/成本/策略中心 → 降低耦合。

### Phase 3：平台化运营期

控制面与执行面分离 + Worker 集群 + Tool 服务化 + 全面多租户治理。

---

## 十七、最小生产落地清单（10 项）

1. 统一 `AgentRequest`、`ToolCall`、`ToolResult`、`DecisionTrace` 对象模型
2. 独立 **LLM Gateway**，不要把模型调用散落在业务代码里
3. 显式**状态机编排**，而不是把循环写死在 `while` 里
4. **Tool Registry** 带 schema、权限、幂等、风险等级
5. Session 与步骤级 **checkpoint** 持久化
6. Trace、Metrics、Audit **三类观测**并存
7. 高风险动作具备**审批与补偿机制**
8. 输入预算、步骤预算、模型预算**三类成本控制**
9. 同步请求与异步任务**分层处理**
10. 至少一条可验证的**降级路径**

---

## 十八、常见工程误区

| 误区 | 修正 |
|------|------|
| 把 Prompt 当作主逻辑 | 主逻辑在状态机、策略、Tool 契约上 |
| 把 Tool 当函数库 | 每个 Tool 有 schema、权限、超时、幂等 |
| 盲目追求多 Agent | 单 Agent + 工具编排能解决就别拆 |
| 不做降级体系 | 异常时能切回规则/模板/人工 |
| 没有恢复能力 | 超长任务必须支持 checkpoint + resume |

---

## 十九、结语

Harness Engineering 的本质：**把"模型能力"变成"可运营能力"**。

从 REPL 到生产，真正的门槛从来不是"是否接入了最强模型"，而是：是否能约束模型、治理 Tool、恢复会话、解释决策、控制成本、在高峰期稳定运行。

**一句话总结：** 它不是让模型更像人，而是让模型驱动的系统更像一套真正可治理的生产基础设施。

---

> **推荐代码仓库结构：**
>
> ```
> agent-platform/
> ├── agent-api              # 对外接入层
> ├── agent-runtime          # 编排状态机
> ├── agent-gateway          # LLM Gateway
> ├── agent-tool-bus         # Tool 执行总线
> ├── agent-policy-center    # 策略、审批、权限
> ├── agent-memory           # 会话记忆与上下文装配
> ├── agent-observability    # trace / metrics / audit
> ├── agent-worker           # 异步执行 worker
> └── agent-console          # 运维与治理后台
> ```
