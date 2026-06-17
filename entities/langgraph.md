---
title: LangGraph
created: 2026-06-04
updated: 2026-06-04
type: entity
tags: [agent-framework, architecture, workflow, agent]
sources: [知识库/技能文档/2026-06-04-LangGraph-RAG-Memory-MCP企业级AI助手架构.md]
confidence: high
---

# LangGraph

LangGraph 是 LangChain 推出的状态编排框架，专为构建**可循环、可分支、可审核、可恢复**的 Agent 工作流而设计。核心思想是**状态中心化**——以 State 为系统唯一数据流转载体。

## 四层架构

| 层 | 职责 |
|----|------|
| 接入层 | 多协议承接流量，安全鉴权与风控 |
| 编排层 | LangGraph 中心化 State 管控全链路状态与分支 |
| 能力层 | RAG 流水线、三层分级记忆、MCP 标准化工具、合规引擎 |
| 存储层 | Milvus、PostgreSQL、Redis 分层存数 |

## 核心编排节点

1. 记忆检索节点
2. 查询预处理节点
3. 路由决策节点
4. RAG 检索节点
5. Agent 推理节点
6. 工具执行节点
7. 合规审核节点
8. 结果输出节点

## MCP 协议集成

MCP（Model Context Protocol）解决传统工具集成 "N 对 N 耦合" 问题，支持：
- **彻底解耦**：工具服务端与客户端分离
- **动态发现**：Agent 启动时自动扫描可用工具
- **安全隔离**：MCP 服务独立部署，物理隔离
- **统一治理**：所有工具调用统一接入、监控、容错、审计

## 生产级特性

- **容错降级**：检索失败→关键词匹配，工具超时→熔断，LLM 异常→兜底应答
- **可观测**：LangSmith + OpenTelemetry 链路追踪，Prometheus + Grafana 指标监控
- **多级记忆**：短期记忆（滑动窗口）+ 长期记忆（持久化）+ 实体记忆（知识图谱）

## 相关页面

- [[hermes-agent]] — Hermes Agent 框架对比
- [[openclaw]] — OpenClaw Harness 编排 vs LangGraph 编排
- [[myharness]] — 自研 Harness 架构对比
- [[deerflow-2.0]] — 字节 Super Agent Harness
