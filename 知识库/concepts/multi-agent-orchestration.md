---
title: 多 Agent 编排
created: 2026-07-21
updated: 2026-07-21
type: concept
tags: [orchestration, agent-framework, harness, production]
confidence: medium
sources:
  - 多Agent编排云原生高并发 (07-13)
  - 深入理解Deep-Agents-Subagents (07-07)
  - 主流Harness实现对比 (06-10)
  - 拆完WorkBuddy (07-15)
  - 港大开源AgentSpace (07-08)
  - AgentScope 2.0深度解读 (06-09)
  - Polynoia (06-14)
---

# 多 Agent 编排

多 Agent 编排是指协调多个 AI Agent 实例协同完成复杂任务的架构模式和方法论。当单个 Agent 的能力边界不足以覆盖任务的复杂度时，通过将任务分解给多个 Specialist Agent，并由 Orchestrator Agent 统一协调，可以显著提升整体系统的能力上限。编排是 [[harness-engineering]] 在分布式场景下的扩展。

## 编排模式

- **主从模式**：一个 Orchestrator Agent 负责任务分解、结果汇总和异常处理，多个 Worker/Subagent 负责具体执行。Deep Agents - Subagents 框架是典型实现。
- **对等协作**：多个 Agent 以对等方式协商分工和结果合并，适用于复杂决策场景。
- **流水线模式**：每个 Agent 处理流水线中的一个环节，前一 Agent 的输出作为后一 Agent 的输入，典型于代码生成流水线。
- **市场模式**：任务被发布到"市场"，Agent 根据自身能力竞标和领取任务，Polynoia 采用群聊方式实现类似机制。

## 开源方案

| 项目 | 特点 |
|------|------|
| **AgentSpace（港大）** | 多 Agent 协作的标准化运行时环境 |
| **AgentScope 2.0** | 分布式多 Agent 框架，支持高并发 |
| **Polynoia** | 微信群聊指挥多 Agent 并行干活 |
| **Deep Agents** | 深入理解 Subagent 分工与协调机制 |
| **WorkBuddy** | 生产级多 Agent 系统的完整形态 |

## 关键挑战

- **通信开销**：Agent 间通信的 token 消耗和延迟管理
- **一致性**：多 Agent 间的状态同步和结果一致性保证
- **容错**：单个 Agent 失败时的任务迁移和恢复
- **安全**：Agent 间权限隔离，防止信息泄露，见 [[agent-governance]]

## 与相关概念的关系

| 概念 | 关系 |
|------|------|
| [[harness-engineering]] | 编排是 Harness 在多 Agent 场景的扩展 |
| [[skill-system]] | 每个 Agent 可以拥有不同的 Skill 组合 |
| [[agent-governance]] | 多 Agent 场景下治理复杂度更高 |
| [[agent-evaluation]] | 评估整个编排系统的协作效率 |
