---
title: Langfuse
created: 2026-07-21
updated: 2026-07-21
type: entity
tags: [evaluation, enterprise, aiops, tool]
sources:
  - 技能文档/2026-07-09-五大Agent可观测性工具对比-MLflow-Langfuse-LangSmith-Phoenix-Braintrust.md
  - 技能文档/2026-07-14-Langfuse-RAGAS生产级可观测评测基建.md
  - 技能文档/2026-06-23-Langfuse-开源LLM工程平台-监控评估调试.md
confidence: medium
---

# Langfuse

## 概述

Langfuse 是**开源 LLM 工程平台**，提供三大核心能力：**可观察性（Observability）**、**Prompt 管理（Prompt Management）** 和 **评估（Evaluation）**。它帮助开发者和团队监控、调试和评估 LLM 应用的性能和输出质量。

在 Agent 生态中，Langfuse 充当"监控中枢"角色，可以接入 LangGraph、DeepAgents、Hermes、Claude Code 等各种 Agent 框架的可观测数据，提供统一的追踪、分析和评测平台。

## 关键事实与日期

- **2026-07-09**: 五大 Agent 可观测性工具对比 — MLflow vs Langfuse vs LangSmith vs Phoenix vs Braintrust
- **2026-07-14**: Langfuse + RAGAS 生产级可观测评测基建详解
- **2026-06-23**: Langfuse 开源 LLM 工程平台介绍 — 监控、评估、调试一体
- 开源产品，可自托管
- 支持 Prompt 版本管理和 A/B 测试
- 与 RAGAS 集成提供 RAG 评测能力
- 支持 Trace 级别的 LLM 调用追踪

## 相关文档

- [[langgraph]] — LangGraph Agent 可通过 Langfuse 进行可观测性监控
- [[deep-agents]] — DeepAgents 的执行可被 Langfuse 追踪
- [[hermes-agent]] — Hermes 多角色执行可通过 Langfuse 评测
- [[agentscope]] — 生产级 Agent 框架，Langfuse 是其可观测性方案之一

## 与其它实体的关系

- **可观测性生态**: 与 MLflow、LangSmith、Phoenix、Braintrust 形成五大可观测性工具阵营
- **RAGAS 集成**: Langfuse + RAGAS 组合提供 RAG 应用的端到端评测方案
- **Prompt 管理**: 除了监控外，Langfuse 还提供 Prompt 生命周期管理能力
- **非 Agent 框架**: Langfuse 不是 Agent 执行框架，而是 Agent 的监控和评测基础设施
- **开源+自托管**: 适合对数据隐私有要求的企业场景

## 来源引用

主要来源：五大Agent可观测性工具对比、Langfuse+RAGAS评测基建、Langfuse开源平台介绍等。详见 sources frontmatter。
