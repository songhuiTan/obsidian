---
title: WeKnora — 腾讯开源的 Go RAG 框架
source: https://mp.weixin.qq.com/s/vavL8R2070VoUaxxmxfb8w
author: 站长 polarisxu
platform: 微信公众号 · Go语言中文网
date: 2026-06-16
tags:
  - RAG
  - Go
  - 腾讯
  - 开源框架
  - AI应用
  - 知识管理
  - Agent
  - 技能文档
status: archived
---

# WeKnora — 腾讯开源的 Go RAG 框架

> 16.2K Stars，微信对话开放平台核心技术框架，Go 编写的生产级 RAG 知识管理框架。

**GitHub:** https://github.com/Tencent/WeKnora

---

## 核心数据一览

| 维度 | 数据 |
|------|------|
| Stars | 16,200 |
| 后端 | Go 1.24 |
| 前端 | Vue.js 3 + Vite |
| 文档格式 | PDF/Word/Excel/PPT/HTML/Markdown/图片/音频/CSV/JSON |
| 向量数据库 | pgvector / Qdrant / Milvus / Weaviate / Elasticsearch |
| LLM Provider | OpenAI / DeepSeek / Qwen / 智谱 / Gemini / Ollama 等 13+ |
| IM 集成 | 企业微信 / 飞书 / 钉钉 / Slack / Telegram / Mattermost |
| Agent 工具 | MCP 协议 / Skill 系统 / 数据分析 / 网络搜索 |
| 部署方式 | Docker Compose / Kubernetes / 离线 |

## 两种问答模式

**快速问答（RAG 流水线）：**
```
用户提问 → 查询重写 → 混合检索（向量+关键词）→ RRF 融合 → Reranking → LLM 生成
```

**智能推理（ReACT Agent）：**
```
用户提问 → Agent 分析 → 检索知识 → 调用 MCP 工具 → 网络搜索 → 反思推理 → 多轮迭代 → 最终回答
```

## 架构：四层分层

```
Handler 层  — Gin + Swagger REST Controllers
Service 层  — Session / Knowledge Base / Agent Engine (ReACT + MCP)
Repository 层 — GORM 类型安全数据访问抽象
Infrastructure 层 — PostgreSQL / Redis + Asynq / Vector Store / Neo4j (GraphRAG)
```

**依赖注入**: `uber-go/dig` — 每层只依赖接口，切换向量数据库只需换 Provider，业务代码零修改。

**异步任务**: Asynq + Redis — 比 Python Celery 轻量（无需额外消息代理），类型安全，goroutine 天然高并发。

## RAG 流水线

### 文档摄取
```
上传文档 → 存储后端 → Asynq 入队 → DocReader (Python gRPC)
                                          │
                    ←—— 文档解析 → 父子分块 → Embedding → 向量存储
```
**DocReader** 是独立的 Python gRPC 服务（PDF/PPT/OCR 生态更成熟），Go 做业务编排，Python 做文档解析——务实的混合架构。

### 父子分块策略（Parent-Child Chunking）
- **子块**（小粒度）用于精确匹配语义
- **父块**（大上下文）用于提供完整上下文
- 解决了传统固定大小分块"太碎丢上下文 / 太粗匹配不准"的问题

### 混合检索
两路并行检索 + RRF 融合 + Reranking + MMR 去重：
- BM25（gojieba 中文分词）关键词检索
- Dense 向量语义检索
- Go `errgroup` 天然并行，goroutine 调度远轻于 Python 协程

## Agent 引擎

### ReACT 模式
```
用户提问 → Agent 思考 → 调用工具（知识检索/MCP/网络搜索）→ 反思 → 继续或回答
```

### 并行工具调用（v0.3.6）
Go `errgroup` 实现 LLM 返回多个工具调用时并行执行，适合同时检索多个知识库或调用多个 MCP 工具。

### MCP 集成
`mark3labs/mcp-go` v0.43.0，支持 stdio / SSE / streamable-http 三种传输模式，自动发现工具并注册到 Agent 工具表，支持自动重连。

## GraphRAG：Neo4j 知识图谱
- 文档摄取时 LLM 提取实体和关系写入 Neo4j
- 查询时先从知识图谱找到相关实体，沿关系扩展获取完整上下文
- 价值：**跨文档关联**，传统 RAG 只能检索片段，GraphRAG 可沿关系图谱找到完整信息链

## 多租户架构
- 数据隔离：所有查询自动注入租户条件
- 共享空间：跨成员共享知识库和 Agent
- 安全：AES-256-GCM API 密钥静态加密、RBAC（Admin/Editor/Viewer）、OIDC 企业 SSO、SSRF 防护

## 6 大 IM 原生集成

| 平台 | SDK | 特色 |
|------|-----|------|
| 企业微信 | WebSocket/Webhook | 微信生态原生集成 |
| 飞书 | larksuite/oapi-sdk-go/v3 | 数据源自动同步 |
| 钉钉 | dingtalk-stream-sdk-go | AI Card 流式输出 |
| Slack | slack-go/slack | 线程会话模式 |
| Telegram | webhook/长轮询 | 流式 editMessageText |
| Mattermost | 适配器模式 | 线程会话 |

每种平台都用 Go SDK 直连，而非 Webhook 转发——延迟更低，错误处理更直接。

## Go 在 RAG 场景的优势

| 维度 | 优势 |
|------|------|
| **并发** | goroutine（2KB）vs Python 协程（1MB+），调度开销低很多 |
| **内存** | 512MB 容器稳定运行，Python 方案通常需要 2-4GB |
| **部署** | 单二进制 + 配置文件，更小镜像、更快启动、更简单 CI/CD |
| **类型安全** | 编译时类型检查，减少复杂 RAG 流水线中的运行时错误 |

## 快速部署

```bash
# 最小部署
docker compose up -d

# 完整部署（含 Neo4j GraphRAG）
docker compose --profile full up -d

# Kubernetes
helm install weknora ./helm/weknora
```

## 与 Hermes 的关联思考

WeKnora 值得关注的几个设计点：

1. **DocReader 混合架构** — Go 编排 + Python 文档解析的务实拆分思路，与 Hermes 的插件化设计有相通之处
2. **MCP 协议集成** — 与 Hermes 的 MCP 客户端一致，可参考其多传输模式支持
3. **IM 原生集成** — 6 大 IM Go SDK 直连，比 Webhook 转发的低延迟架构设计
4. **父子分块策略** — 解决 RAG 中"精度 vs 上下文"矛盾的具体实践
5. **GraphRAG 多跳关联** — 对于跨文档知识管理有参考价值
