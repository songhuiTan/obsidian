---
title: "你的 Agent 终于能碰数据库了：Google 开源的 mcp-toolbox 打通了 20+ 种数据引擎"
author: "openvoid / OpenMCP"
date: "2026-07-01"
source: "https://mp.weixin.qq.com/s/N3Ll-_lKClDCvqAjzbWo0Q"
tags: [MCP, 数据库, Google, mcp-toolbox, DBHub, 开源工具, Agent工具生态]
---

# 你的 Agent 终于能碰数据库了：Google 开源的 mcp-toolbox 打通了 20+ 种数据引擎

> Google 开源 mcp-toolbox（15.8k ⭐），一个 MCP 服务器，让 Claude Code、Codex、Gemini CLI 等 AI 助手直接连接数据库——查表结构、跑 SQL、参数化查询，全在对话中完成。

## 两层用法

### 第一层：开箱即用（Prebuilt 模式）
一条命令即用，零配置：
```bash
npx @toolbox-sdk/server --prebuilt=postgres --stdio
```
AI 助手立刻获得 list_tables、get_schema、execute_sql 等标准工具。适合开发调试、快速探索。

### 第二层：自定义工具框架（生产环境）
写一个 `tools.yaml`，把 SQL 模板化、参数化，精确控制 Agent 能跑什么查询、不能碰哪些表。

典型场景：
```yaml
# 定义一个「按用户名搜索酒店」的工具
# Agent 只能执行你写好的那条 SELECT，没法 DROP TABLE
```
**核心设计理念：你定义好工具边界，Agent 在边界内执行。** 连接池、IAM 鉴权、OpenTelemetry 全链路追踪都帮你管了。

> ⚠️ 生产环境务必用 tools.yaml 模式，别让 LLM 自由跑 SQL。

## 支持数据库（20+）

PostgreSQL / MySQL / MariaDB / SQL Server / Oracle / MongoDB / Redis / Elasticsearch / CockroachDB / ClickHouse / Snowflake / Neo4j / Trino / AlloyDB / BigQuery / Cloud SQL / Spanner / Firestore……

## Skills 生成功能

`toolbox skills-generate` — 一键把定义好的 toolset 导出成 Agent Skill 包，装进 Gemini CLI 就能用。从数据库工具到可分发的能力包，一步到位。

## 配置方式

MCP 客户端配置：
```json
{
  "mcpServers": {
    "toolbox": {
      "type": "http",
      "url": "http://127.0.0.1:5000/mcp"
    }
  }
}
```

SDK 覆盖 Python、JS/TS、Go、Java，各语言都接入了主流 Agent 框架（LangChain、LlamaIndex、Genkit、ADK）。

## 同类工具对比

| 工具 | Star | 定位 | 特点 |
|------|------|------|------|
| **mcp-toolbox** | 15.8k | 全功能数据库 MCP 框架 | 20+ 引擎、4语言SDK、Skills生成、自定义工具框架、UI 测试界面、可观测性 |
| **bytebase/dbhub** | 3k | 轻量零依赖直连 | 单个二进制、零外部依赖、专注安全读写数据库 |
| **centralmind/gateway** | 528 | 自然语言翻译层 | 日常问法自动转 SQL，对 LLM 深度优化 |

> mcp-toolbox 做"全功能"，dbhub 做"轻量直连"，gateway 做"自然语言翻译"。不互斥，按需选择。生产环境倾向 mcp-toolbox。

## 相关资源

- GitHub：https://github.com/googleapis/mcp-toolbox
- 官方文档：https://mcp-toolbox.dev/documentation/introduction
- Gemini CLI 扩展：https://github.com/gemini-cli-extensions/mcp-toolbox
- Google Cloud 托管版 MCP Server：https://cloud.google.com/blog/products/databases/managed-mcp-servers-for-google-cloud-databases
