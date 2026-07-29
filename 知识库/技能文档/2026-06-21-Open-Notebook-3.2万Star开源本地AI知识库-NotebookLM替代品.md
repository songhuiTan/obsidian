# Open Notebook — 3.2万 Star 的开源本地 AI 知识库，Notebook LM 替代品

> **来源：** 公众号「极客之家」@丛林
> **发布时间：** 2026-06-21
> **原文：** https://mp.weixin.qq.com/s/ACLLu6zMlnEgXcte7v1JYQ
> **GitHub：** https://github.com/lfnovo/open-notebook
> **Star：** 32K · **License：** MIT

## 概述

Open Notebook 是一个完全开源的本地 AI 知识库，对标 Google Notebook LM 但更强大——支持 18 家 AI 提供商自由切换、多格式文档上传、RAG 问答、播客生成，以及 **MCP 集成**。`docker-compose up -d` 即可运行。

## 技术栈

| 层 | 选型 |
|---|------|
| 后端 | Python + FastAPI |
| 前端 | Next.js + React |
| 数据库 | SurrealDB |
| AI 抽象层 | Esperanto（统一 18 家 API） |
| 部署 | `docker-compose.yml` 一键跑 |
| 端口 | 8502（Web）/ 5055（REST API）|

## 核心功能

### 1. 18+ AI 模型自由切换

支持 OpenAI、DeepSeek、Claude、Ollama（本地）、Groq 等 18 家提供商。在 UI 中配置 API Key → Test → Sync Models → 勾选可用模型。三个槽位分别指定 Chat Model、Transformation Model、Embedding Model。

推理模型也支持（DeepSeek-R1 / Qwen3 思考链）。接 Ollama 本地模型可做到**全链路不经过外部 API**。

### 2. 多格式输入

支持拖拽上传：PDF、音频文件、网页链接、TXT、PPT、Word 文档。后台自动做向量化和索引。

### 3. RAG 对话 + 来源标注

基于 RAG 流程：向量检索 → 片段 → LLM 生成。回答标注引用来源，可追溯到具体段落。每个资料来源旁有"小灯泡"——切换使用全文还是摘要。

### 4. 播客生成（1-4 个说话人）

Notebook LM 只能两人+不可定制脚本，Open Notebook 可以：
- 1 到 4 个说话人
- 每个说话人独立设置音色、角色、语言
- 写 Episode Profile 定主题/风格/时长
- TTS 后端：ElevenLabs / OpenAI TTS / Google TTS / Edge-TTS（免费）

### 5. 全文 + 语义双模搜索

传统全文搜索（关键词精确匹配）+ 向量语义搜索（"用户登录"能匹配"鉴权模块"）。搜索范围跨所有笔记本。

### 6. 内容转换 / AI 笔记

- **Insights 面板：** 自动生成 Dense Summary
- **Content Transformations：** 自定义转换规则（如提取所有竞品价格到表格）
- **AI 笔记：** 手动写或 AI 根据资料生成，与来源资料关联标记

### 7. REST API + MCP 集成

- **REST API（5055 端口）：** 全部功能可通过代码调用（上传、建笔记本、发起聊天、生成播客）
- **MCP Server 模式：** 可接入 Claude Desktop 或 VS Code，在 IDE 中直接搜索/引用知识库内容

## 部署方式

```bash
# 唯一前提：Docker Desktop
curl -o docker-compose.yml https://raw.githubusercontent.com/lfnovo/open-notebook/main/docker-compose.yml
# 修改加密密钥：OPEN_NOTEBOOK_ENCRYPTION_KEY=change-me-to-a-secret-string
docker compose up -d
# 浏览器打开 http://localhost:8502
```

## 当前局限

- **单用户** — 不支持多人协作
- **引用功能** — 比 Notebook LM 的原文高亮定位弱
- **大文件处理速度** — 几百页 PDF 纯 CPU Embedding 较慢（取决于机器配置）

## 战略分析

### 与现有工具链的对照

| 维度 | Open Notebook | 我当前的方案 |
|------|--------------|------------|
| RAG 引擎 | 内置，UI 友好 | ChromaDB（CLI 驱动）|
| AI 提供商 | 18 家，UI 配置 | Hermes provider 配置 |
| MCP 集成 | 支持 MCP Server | Hermes 原生 MCP |
| 播客生成 | **独有**，4 说话人 | — |
| 部署方式 | Docker 全家桶 | Python 脚本 + Launchd |
| 内容组织 | Notebook 分区 | Obsidian 知识库 |
| API 接口 | REST + MCP | Hermes Gateway + cron |
| 数据隐私 | 全本地 | 全本地 |

### 价值定位

Open Notebook 与 kb-builder 在**功能上高度重叠**，但定位不同：

- **kb-builder**：面向 Claude Code 的 Skill（CLI 驱动、开发者工具链）
- **Open Notebook**：面向普通用户的 Web UI（可视化操作、播客生成）

它最强的差异化能力是**播客生成**（4 说话人 + 可定制脚本 + TTS 多引擎），这是当前工具链条中缺失的一环。

### 整合可能性

1. **MCP 接入**：Open Notebook 可作为 MCP Server 接入 Hermes，让知识库检索与当前的工作流互补
2. **播客能力**：如果需要将归档内容转为音频，这是一个现成的方案
3. **Notebook 分区**：按项目/主题分区管理知识库的思路值得参考

### 同类对比

| 项目 | Star | 特点 |
|------|------|------|
| Open Notebook | 32K | MCP 集成、播客生成、18 模型 |
| Trove AI + Obsidian | — | 与 Obsidian 深度绑定 |
| kb-builder | 新项目 | Claude Code Skill、双 Repo 分离 |

## 相关文档

- [[技能文档/2026-06-14-干掉NotebookLM-Trove-AI开源项目+Obsidian-本地AI知识库]]
- [[技能文档/kb-builder — 一键搭建AI知识库（Claude Code Skill + ChromaDB + MCP）]]
- [[技能文档/2026-06-20-从原始到Agentic——8种RAG架构深度解析与生产实践指南]]
