---
title: "智能 OnCall Agent 项目 — 大模型Agent开发实战"
source: "nullbody笔记"
source_url: "https://mp.weixin.qq.com/s/6crIyIrfWOSoXYmxeDTRYw"
date: "2026-05-14"
tags: [AI, Agent, AIOps, Go, Eino, Milvus, RAG, DeepSeek, 开源]
---

# 智能 OnCall Agent 项目 — 大模型Agent开发实战

> 来源：nullbody笔记｜微信公众号 · 2026-05-14
> 原文：[智能 OnCall Agent 项目 ｜ 大模型Agent开发实战](https://mp.weixin.qq.com/s/6crIyIrfWOSoXYmxeDTRYw)
> 项目地址：[gofish2020/OncallAgent](https://github.com/gofish2020/OncallAgent)

## 概述

基于 AI 的企业级运维自动化助手项目，解决传统 OnCall 值班中人工值守和排查问题的低效痛点。整合了**知识库 Agent**、**对话 Agent**、**运维 Agent** 三大核心能力，实现问题自动应答和故障智能排查的一体化服务。后端采用 **Go + GoFrame + Eino AI 框架**（字节开源），向量数据库使用 Milvus，大模型使用 DeepSeek，本地向量模型使用 Ollama。

## 核心架构

### 技术栈

| 层面 | 选型 | 用途 |
|------|------|------|
| 后端语言 | Go | 高性能服务 |
| Web 框架 | GoFrame | HTTP 路由、中间件 |
| AI 框架 | Eino（字节开源） | Agent 图编排、节点编排 |
| 大模型 | DeepSeek V3（思维 + 快速双模型） | Planner、推理、对话 |
| 向量模型 | Ollama nomic-embed-text / 阿里百炼 text-embedding-v4 | 知识库向量化 |
| 向量数据库 | Milvus | 知识检索（RAG） |
| 日志系统 | 腾讯云 CLS（通过 MCP 协议） | 日志查询 |
| 前端 | 原生 JavaScript + HTML/CSS（vibe coding 生成） | Web 界面 |

### API 端点

| 端点 | 方法 | 功能 |
|------|------|------|
| `/api/chat` | POST | 快速对话（非流式） |
| `/api/chat_stream` | POST | 流式对话（SSE） |
| `/api/upload` | POST | 文件上传 |
| `/api/ai_ops` | POST | AI 运维分析 |

## 三大 Agent Pipeline

### 1. 对话 Agent（Chat Pipeline）

**图结构**（Eino Graph）：

```
START
  ├── InputToRag → MilvusRetriever ──┐
  └── InputToChat ─────────────────────┤
                                       ↓
                               ChatTemplate → ReactAgent → END
```

- **InputToRag**：从用户消息提取 query 字符串
- **MilvusRetriever**：向量检索，从知识库获取 Top-3 相关文档
- **InputToChat**：构建上下文（历史消息 + 当前时间）
- **ChatTemplate**：组装提示词模板
- **ReactAgent**：LLM 推理 + 工具调用（Function Calling）

### 2. 知识库构建 Agent（Knowledge Index Pipeline）

**处理流程**：Markdown 文档 → 按标题/段落分割 → Ollama Embedder 向量化 → 存入 Milvus

### 3. 运维 Agent（Plan-Execute-Replan）

**核心模式**：将复杂任务分解为规划、执行、重规划的循环过程。

| 阶段 | 组件 | 模型 | 功能 |
|------|------|------|------|
| 规划 | Planner | DeepSeek V3 思维模型 | 生成处理计划 |
| 执行 | Executor | DeepSeek V3 Quick | 调用工具集执行 |
| 重规划 | Replanner | DeepSeek V3 思维模型 | 根据执行结果调整计划 |

**迭代限制**：最多 20 轮迭代。

**可用工具集**：

| 工具 | 功能 |
|------|------|
| `query_prometheus_alerts` | 查询 Prometheus 活跃告警 |
| `query_internal_docs` | 查询内部知识库（RAG） |
| `query_log` | 查询日志（通过 MCP 连接腾讯云 CLS） |
| `query_metrics_alerts` | 查询指标和告警 |
| `get_current_time` | 获取当前时间 |
| `MysqlCrud` | 数据库操作（需用户确认） |

## 关键技术点

### RAG 检索增强生成

| 步骤 | 实现 | 说明 |
|------|------|------|
| 检索 | Milvus 向量相似度搜索 | 从知识库找相关文档 |
| 增强 | 文档嵌入到 Prompt | 提供 LLM 上下文 |
| 生成 | LLM 基于上下文回答 | 准确的知识问答 |

### 会话内存管理

- **SimpleMemory**：基于会话 ID 的内存缓存
- **滑动窗口**：维护最近 6 条消息（3 轮对话）
- **线程安全**：使用 `sync.Mutex`

### 双模型策略

| 模型 | 用途 | API |
|------|------|-----|
| `deepseek-reasoner`（V3.1 思维） | Planner、重规划器 | `ds_think_chat_model` |
| `deepseek-chat`（V3 Quick） | 对话 Agent、Executor | `ds_quick_chat_model` |

将"深度思考"和"快速响应"分离，平衡推理质量与成本。

## 启动方式

```bash
# 下载代码
git clone https://github.com/gofish2020/OncallAgent.git

# 启动 Docker 服务（Milvus 等）
cd ./manifest/docker && docker-compose up -d

# 配置 config.yaml（复制模板并填写 API Key）
cp ./manifest/config/config.example.yaml ./manifest/config/config.yaml

# 启动后端
go run main.go   # 服务在 http://localhost:6872

# 启动前端
cd SuperBizAgentFrontend && ./start.sh   # 界面在 http://localhost:8000
```

## 关键洞察

- **Eino 框架**：字节开源的新一代 AI Agent 框架，图编排方式构建 Agent Pipeline，节点间通过输出/输入类型约束串联。本文提供了完整的 Chat、Knowledge Index、Plan-Execute-Replan 三种图结构参考，是学习 Eino 开发的良好范例。
- **Plan-Execute-Replan 模式**：相比简单的 ReAct，新增了显式的重规划阶段，更适运维场景中需要多步推理、动态调整的复杂任务。
- **MCP 集成日志**：通过 MCP 协议对接腾讯云 CLS 日志服务，展示了 AI Agent 如何通过标准协议与已有基础设施集成。
- **滑动窗口内存**：简单的 6 消息窗口管理，适合会话式场景，但缺乏持久化——生产环境需要数据库支持。
- **成本优化**：本地 Ollama 向量模型 + DeepSeek 双模型策略（思维模型仅 Planner 使用），体现了成本敏感性设计。

## 归档日志

- 2026-05-14 归档
