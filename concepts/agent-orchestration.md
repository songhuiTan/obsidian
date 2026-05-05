---
title: Agent 编排 (Agent Orchestration)
created: 2026-05-05
updated: 2026-05-05
type: concept
tags: [agent, workflow, architecture, trend]
sources:
  - raw/articles/mit-techreview-10-things-ai-2026.md
  - 知识库/技能文档/OpenDeepCrew - AI Agent团队编排服务器-2026-03-27.md
  - 知识库/技能文档/DeerFlow 2.0 - 字节Super Agent Harness-2026-03-27.md
confidence: high
---

# Agent 编排 (Agent Orchestration)

## 定义

Agent 编排指多个 AI Agent 协同工作以完成复杂目标的能力。MIT Technology Review 将其列为2026年AI最重要的趋势之一——"第一波AI Agent只能单独行动，下一波是能协作的Agent团队"。^[raw/articles/mit-techreview-10-things-ai-2026.md]

## 现有实现

知识库中已有相关的开源实现：

- [[opendeepcrew]] — OpenDeepCrew：AI Agent 团队编排服务器
- [[deerflow-2.0]] — DeerFlow 2.0：字节跳动 Super Agent Harness
- [[myharness]] — MyHarness：从说明书到引擎的 Agent 编排方案
- [[openspace]] — OpenSpace：核心机制分析

## 关键技术维度

1. **任务分解** — 将复杂目标拆解为子任务
2. **Agent间通信** — 共享上下文和中间结果
3. **冲突解决** — 多个Agent决策冲突时的仲裁机制
4. **工具调用** — Agent自主选择和执行工具
5. **记忆共享** — Agent团队共享的经验和知识

## 与知识蒸馏的关系

蒸馏后的轻量模型使得在成本受限的环境中部署多Agent团队成为可能——每个Agent占用更少的计算资源，团队规模可以更大。
- 参见：[[知识蒸馏]]
