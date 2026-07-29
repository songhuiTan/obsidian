---
title: "MCP（Model Context Protocol）"
created: 2026-07-21
updated: 2026-07-21
type: concept
tags: [mcp, tool, agent-framework, integration]
sources:
  - "技能文档/2026-06-28-Function-Call-MCP-工具治理.md"
  - "技能文档/2026-07-01-Google-mcp-toolbox-打通20种数据引擎.md"
  - "技能文档/2026-06-04-LangGraph-RAG-Memory-MCP企业级AI助手架构.md"
  - "技能文档/2026-06-02-用Python+MCP搭AIOps运维助手.md"
  - "技能文档/2026-07-17-AI-Agent-MCP-LangChain无人机租赁系统.md"
  - "技能文档/2026-05-25-Harness-Engineering-从零搭建-Rule-Skill-Sub-Agent完整路径.md"
confidence: medium
---

# MCP（Model Context Protocol）

MCP（Model Context Protocol）由 Anthropic 提出，是 AI 应用与外部系统之间的开放连接协议。它将 "N 个 AI Host × M 个系统" 的复杂连接关系简化为标准化接口。

## 核心能力

MCP Server 暴露三类核心能力：

| 能力 | 说明 | 示例 |
|------|------|------|
| Tools | 可被调用的执行能力 | `ticket.create`, `database.query` |
| Resources | 可读取的上下文数据 | `file:///README.md`, `git://.../OrderService.java` |
| Prompts | 参数化模板或工作流入口 | `review_pull_request`, `analyze_incident` |

## Function Call 与 MCP 的关系

| 维度 | Function Call | MCP |
|------|---------------|-----|
| 核心目标 | 模型结构化表达工具调用 | 标准化 AI 应用与外部服务连接 |
| 主要边界 | 模型 ↔ Agent Runtime | MCP Client ↔ MCP Server |
| 工具发现 | 由应用提供定义 | 可通过协议列出服务端能力 |
| 自动安全 | 否 | 否 |

完整链路：模型 → Function Call → Tool Gateway → 校验 → MCP Client → MCP Server / 业务系统。

## 企业级应用

**Google MCP Toolbox**：打通 20 种数据引擎，提供统一的数据访问层。

**AIOps 运维助手**：Python + MCP 搭建智能运维系统，将监控告警、故障诊断等能力标准化。

**无人机租赁系统**：[[langgraph]] + MCP + LangChain 构建的复合 Agent 系统。

**MCP 不是**：大模型、Agent Loop、RAG 算法、工作流引擎、权限中心、API Gateway。MCP 标准化"如何通信"，但不替企业完成安全治理。

## 工具治理

MCP 需要与工具治理体系配合使用，包括注册、发现、权限、安全、版本、执行、审计和下线等工程环节。[[hermes-agent]] 的 MCP 集成实践表明，MCP Server 的标准化接口设计大幅降低了工具接入成本。
