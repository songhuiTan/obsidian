---
title: AgentScope
created: 2026-07-21
updated: 2026-07-21
type: entity
tags: [agent-framework, enterprise, orchestration]
sources:
  - 技能文档/AgentScope 2.0深度解读-阿里达摩院生产级多智能体框架.md
  - 技能文档/AgentScope框架浅析与最佳实践-从框架内核到生产级多智能体落地.md
  - 技能文档/AgentScope-Java-2.0-无状态引擎和AgentState详解.md
  - 技能文档/2026-06-04-AgentScope-Java-2.0-打造分布式企业级智能体底座.md
confidence: medium
---

# AgentScope

## 概述

AgentScope 是阿里达摩院出品的**生产级多智能体框架**，定位为企业级分布式 AI Agent 平台。其 2.0 版本引入了 Java 无状态引擎和分布式底座，支持大规模多 Agent 部署和协作。

AgentScope 的设计重点在于生产环境的可靠性、可扩展性和性能。它提供了完整的 Agent 生命周期管理、分布式通信、状态持久化等企业级能力，与学术导向或轻量级 Agent 框架形成鲜明对比。

## 关键事实与日期

- **2026-06-09**: AgentScope 2.0 深度解读 — 多智能体框架全面分析
- **2026-06-24**: AgentScope 框架浅析与最佳实践
- **2026-06-23**: AgentScope-Java-2.0 无状态引擎与 AgentState 详解
- **2026-06-04**: AgentScope-Java-2.0 分布式企业级智能体底座
- Java 2.0 版本引入无状态引擎（Stateless Engine），提升水平扩展能力
- 分布式底座支持大规模多 Agent 部署
- 生产级特性：状态管理、错误恢复、监控告警

## 相关文档

- [[deep-agents]] — LangChain 的深度自主 Agent，不同流派的多 Agent 设计
- [[hermes-agent]] — 开源多角色 Agent 框架
- [[langfuse]] — LLM 可观测性平台，可用于 AgentScope 生产监控

## 与其它实体的关系

- **vs DeepAgents/Hermes**: AgentScope 更强调**生产级部署**和**企业级特性**，而后两者更注重 Agent 能力本身
- **Java 生态**: 2.0 版本的 Java 实现使其更容易融入 Java 企业技术栈
- **阿里达摩院**: 学术+工程背景，框架设计严谨
- **分布式底座**: 是 AgentScope 区别于其他 Agent 框架的核心差异化优势
- **无状态引擎**: 支持 Agent 的水平扩展和弹性伸缩

## 来源引用

主要来源：AgentScope 2.0深度解读、框架浅析、Java 2.0无状态引擎与分布式底座等。详见 sources frontmatter。
