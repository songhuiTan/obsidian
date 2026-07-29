---
source: 微信公众号
author: 一飞开源（开源 Star）
url: https://mp.weixin.qq.com/s/TPZAajVznKFyvjjv9ahrSw
date: 2026-06-23
tags: [Langfuse, LLMOps, 可观察性, 提示管理, 评估, 开源]
---

# [开源] 一个开源 LLM 工程平台——Langfuse

> 帮助团队协作开发、监控、评估以及调试 AI 应用

## Langfuse 是什么

Langfuse 是一个 **开源 LLM 工程平台**，帮助团队协作 **开发、监控、评估** 以及 **调试** AI 应用。可在几分钟内 **自托管**，MIT 开源协议。

开源地址：https://github.com/langfuse/langfuse

## 核心特性

### LLM 应用可观察性
为应用插入仪表代码，追踪 LLM 调用及应用中其他相关逻辑（检索、嵌入、代理操作）。检查并调试复杂日志及用户会话。

### 提示管理（Prompt Management）
集中管理、版本控制并协作迭代提示。利用服务器和客户端高效缓存，在不增加延迟的情况下反复迭代。

### 评估
支持 LLM 作为"裁判"、用户反馈收集、手动标注以及通过 API/SDK 实现自定义评估流程。

### 数据集
为评估 LLM 应用提供测试集和基准，支持持续改进、部署前测试、结构化实验，与 LangChain、LlamaIndex 等框架无缝整合。

### LLM 试玩平台（Playground）
用于测试和迭代提示及模型配置的工具，缩短反馈周期。从追踪中发现异常结果时，可直接跳转至试玩平台调整。

### 综合 API
提供 OpenAPI 规格、Postman 集合以及 Python 和 JS/TS 的类型化 SDK，用于驱动定制化的 LLMOps 工作流程。

## 自托管部署

- **本地（Docker Compose）**：5 分钟内运行
  ```bash
  git clone https://github.com/langfuse/langfuse.git
  cd langfuse
  docker compose up
  ```
- **Kubernetes（Helm）**：推荐的生产环境部署方式
- **虚拟机**：Docker Compose 单机部署
- **Terraform 模板**：AWS、Azure、GCP

## 主要集成

| 集成 | 语言/平台 | 描述 |
|------|----------|------|
| SDK | Python, JS/TS | 手动仪表化 |
| OpenAI | Python, JS/TS | 自动仪表化 |
| LangChain | Python, JS/TS | 回调处理器 |
| LlamaIndex | Python | 回调系统 |
| Haystack | Python | 内容追踪 |
| LiteLLM | Python, JS/TS | 100+ LLMs |
| Vercel AI SDK | JS/TS | AI 驱动应用 |
| API | — | OpenAPI 规格 |

### 社区集成
DSPy、AutoGen、CrewAI、Flowise、Langflow、Dify、OpenWebUI、Promptfoo、LobeChat、Ollama、Amazon Bedrock、Google VertexAI & Gemini 等。

## 关键信息

- **开源协议**：MIT
- **GitHub**：github.com/langfuse/langfuse
- **一飞开源**：https://code.exmay.com/
