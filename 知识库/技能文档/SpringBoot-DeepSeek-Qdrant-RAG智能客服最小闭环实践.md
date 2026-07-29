---
source: 微信公众号
title: 2 小时搞定 RAG 智能客服：Spring Boot + DeepSeek + Qdrant 最小闭环实践
author: yeffky
date: 2026-06-21
category: RAG 与知识检索
tags: RAG, Spring Boot, DeepSeek, Qdrant, Java, 智能客服, LangChain4j, Markdown
link: https://mp.weixin.qq.com/s/rw9vxk4-Bl_J60wCy0cnsw
status: 已归档
---

# 2 小时搞定 RAG 智能客服：Spring Boot + DeepSeek + Qdrant 最小闭环实践

> 作者：yeffky | 2026-06-21
> 项目：https://github.com/yeffky/hmdp-ai-agent
> 原文：[微信链接](https://mp.weixin.qq.com/s/rw9vxk4-Bl_J60wCy0cnsw)

---

## 核心概念：RAG 最小闭环

```
写入管线：文档 → 切片 → 向量化 → 存入向量库
检索管线：用户提问 → 向量化 → 相似检索 → 拼接上下文 → LLM 回答
```

两个管线各 3 步，代码量不超过 500 行 Java。

---

## 架构总览

```
┌─────────────────────────────────────┐
│    前端 (Vue.js + Element UI)         │
│    客服悬浮窗 / 知识库管理后台         │
└─────────────────┬───────────────────┘
                  │ HTTP
┌─────────────────▼───────────────────┐
│        Spring Boot 后端               │
│  ┌──────────┐ ┌───────────┐ ┌──────┐ │
│  │ AI Agent │ │ RAG 引擎  │ │ 业务  │ │
│  │ 五层架构  │ │ 摄取+检索 │ │ 服务  │ │
│  └──────────┘ └───────────┘ └──────┘ │
└──────┬──────────────────┬────────────┘
       │                  │
┌──────▼──────┐  ┌───────▼────────┐
│  DeepSeek   │  │ Qdrant + Ollama│
│  LLM API    │  │ 向量库 + Embed │
└─────────────┘  └────────────────┘
```

---

## 写入管线

### Step 1：Markdown 结构感知切片

关键设计——**结构化文档必须用结构化切分**，不能像 LangChain 的 `TextSplitter` 那样按字符一刀切。

**`MarkdownSplitter` 能力：**
- ✅ 按 H2/H3 标题层级切分，保持语义完整
- ✅ 代码块和 Markdown 表格**自动保护**（占位符替换 → 切分 → 还原）
- ✅ 每个切片携带**标题路径**（如 `帮助中心 > 退款政策 > 申请流程`）
- ✅ 段落 → 句子 → 字符的**三级兜底策略**

### Step 2：向量化（bge-m3 via Ollama）

OpenAI 兼容格式，5 行代码可切换 provider。

```java
// OpenAiEmbeddingService.java
public float[] embed(String text) {
    Map<String, Object> body = Map.of("model", "bge-m3", "input", text);
    // POST → http://localhost:11434/v1/embeddings
    ResponseEntity<Map> resp = restTemplate.postForEntity(baseUrl + "/embeddings", body, Map.class);
    // 解析 float[] 返回
}
```

选择 **bge-m3** 的原因：中文 FAQ 类内容检索准确率高，Ollama 一键拉起零配置。

### Step 3：Qdrant 存储（纯 REST API，零 SDK 依赖）

```bash
docker compose up -d  # 20 行 docker-compose.yml
```

```java
// QdrantVectorStore.java
public boolean upsert(List<DocumentChunk> chunks) {
    // POST /collections/{name}/points?wait=true
}
public List<SearchResult> search(float[] queryVector, int topK, double scoreThreshold) {
    // POST /collections/{name}/points/search
}
```

**写入管线串联：** `文档 Markdown → MarkdownSplitter 切片 → bge-m3 向量化 → Qdrant 存储`

---

## 检索管线

### Step 4：检索 + 格式化

```java
public String retrieveAndFormat(String query) {
    float[] queryVector = embeddingService.embed(query);
    List<SearchResult> results = qdrant.search(queryVector, topK, scoreThreshold);
    return formatContext(results);  // → "参考以下信息:\n[1] 文档A: ..."
}
```

**设计决策：** `RetrievalService` 只负责检索，格式化独立出来——方便加缓存、重排序、多轮对话上下文压缩。

### Step 5：Agent 工具化（LangChain4j @Tool）

```java
@Tool("从知识库中检索信息。当用户询问平台规则、使用帮助时调用")
public String searchKnowledge(String query) {
    return retrievalService.retrieveAndFormat(query);
}
```

System Prompt 规定决策逻辑（LLM 自己决定路由，不需要写 if-else）：

| 查询类型 | 路由策略 |
|---------|---------|
| GENERAL_QA | → searchKnowledge 检索知识库 |
| SHOP_INFO | → searchKnowledge（商家信息、营业时间） |
| USER_INFO | → searchKnowledge（账号设置） |
| ORDER_QUERY | → queryMyOrders 查真实订单数据 |
| COMPLAINT | → 安抚情绪，记录反馈 |

---

## 关键坑：DeepSeek 不认 `role=function`

**问题：** LangChain4j 0.31 将工具执行结果序列化为 `role=function`，但 DeepSeek API 只接受 `role=tool / system / user / assistant`，返回 400。

**解决方案：60 行 HTTP 代理**

```java
// DeepSeekProxyController.java
@RequestMapping(value = "/**")
public ResponseEntity<String> proxy(HttpServletRequest request, @RequestBody(required = false) String body) {
    String path = request.getRequestURI().replace("/api/deepseek-proxy", "");
    String targetUrl = realBaseUrl + path + "?" + query;

    // ★ 核心修复：一行正则替换
    if (body != null && body.contains("\"role\":\"function\"")) {
        body = body.replace("\"role\":\"function\"", "\"role\":\"tool\"");
    }

    headers.setBearerAuth(apiKey);
    return restTemplate.exchange(targetUrl, method, new HttpEntity<>(body, headers), String.class);
}
```

**为什么代理方案更好：**
- 工具调用走标准 `role=tool` 协议，Agent 的 @Tool 能力完整恢复
- 对 LangChain4j 和 DeepSeek 都是透明的，两边都不需要改
- MCP、function calling 等后续扩展不再受兼容性限制
- 只需在配置里把 `deepseek.base-url` 指向代理地址

> 这个代理模式在 Java 生态里通用：当你需要在 API 调用链路中插入轻量级中间层，Spring Boot 的 @RestController + RestTemplate 是最轻量的方案。

---

## 效果对比

| 维度 | 普通 LLM 模式 | RAG 增强模式 |
|------|-------------|-------------|
| 商家信息 | 泛泛而谈，可能编造 | 引用知识库原文 |
| 订单查询 | 编造订单号（幻觉） | 通过 Agent Tool 查真实数据 |
| 知识更新 | 必须重新训练/微调 | 重新摄入文档即可 |

---

## 总结

**核心要点：**
- 最小 RAG 闭环 = 写入管线 + 检索管线，代码量不超过 500 行 Java
- Markdown 文档必须用**结构感知切分**——代码块和表格需要占位符保护
- Qdrant REST API 足够好用，不需要 SDK
- DeepSeek 与 LangChain4j 的 role=function 兼容问题用 **HTTP 代理一劳永逸**

**适用场景：** 企业知识库问答（FAQ、帮助中心）、电商客服（商品信息、退换政策）

**扩展方向：** Re-rank 重排序、多轮对话上下文压缩、增量更新 Webhook 触发
