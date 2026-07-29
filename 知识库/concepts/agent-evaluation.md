---
title: Agent 评测体系
created: 2026-07-21
updated: 2026-07-21
type: concept
tags: [evaluation, agent-framework, production, methodology]
confidence: medium
sources:
  - Agent评测体系从demo到生产 (07-02)
  - 基于顶级Agent的Harness评测方案 (06-05)
  - 用Agent评测管理AI Coding (05-07)
  - 五大Agent可观测性工具对比 (07-09)
  - AI-Agent生产级面试18问 (07-07)
---

# Agent 评测体系

Agent 评测体系是衡量 AI Agent 在真实生产环境中表现的方法论和工具集。不同于传统的 NLP 评测（BLEU、ROUGE 等），Agent 评测关注的是 Agent 在多轮交互、工具调用、任务完成度、可靠性等维度的综合表现。它与 [[harness-engineering]] 的质量门禁深度绑定，是 [[agent-governance]] 的数据基础。

## 评测维度

- **任务完成度**：Agent 是否成功完成了用户指定的任务，包括对复杂多步任务的拆解和执行率。
- **工具调用准确性**：Agent 是否正确选择了工具、传入了正确的参数、处理了工具返回的结果。
- **鲁棒性**：Agent 在异常输入、上下文截断、工具超时等情况下的表现。
- **效率**：完成任务所需的 token 消耗、调用次数、响应时间。
- **可观测性**：Agent 的决策过程是否可追溯、可审计。参考五大可观测性工具对比。

## 评测工具生态

五大 Agent 可观测性工具对比覆盖了 MLflow、Langfuse、LangSmith、Phoenix、Braintrust 等主流方案，各有侧重：

| 工具 | 特点 |
|------|------|
| MLflow | 开源、与 ML 生态深度集成 |
| Langfuse | 专注 LLM 可观测性，社区活跃 |
| LangSmith | LangChain 生态原生支持 |
| Phoenix | Arize 出品，侧重生产监控 |
| Braintrust | 企业级评测管理平台 |

## 与相关概念的关系

| 概念 | 关系 |
|------|------|
| [[harness-engineering]] | 评测是 Harness 质量门禁的核心组件 |
| [[skill-system]] | 每个 Skill 的产出都需要被评测 |
| [[loop-engineering]] | Inspector 本质上是评测机制 |
| [[agent-governance]] | 评测结果是治理决策的数据依据 |
