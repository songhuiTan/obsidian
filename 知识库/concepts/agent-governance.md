---
title: Agent 治理
created: 2026-07-21
updated: 2026-07-21
type: concept
tags: [governance, agent-framework, production, enterprise]
confidence: medium
sources:
  - Agent治理-Hook堵LLM偷懒越权失忆 (07-16)
  - Function Call-MCP-工具治理 (06-28)
  - Loopany多用户定时循环调度 (07-18)
  - 多Agent编排云原生高并发 (07-13)
  - AI-Agent生产级面试18问 (07-07)
---

# Agent 治理

Agent 治理是确保 AI Agent 在生产环境中安全、合规、可控运行的管理体系。核心挑战在于：LLM 本身存在偷懒（lazy）、越权（privilege escalation）、失忆（context forgetting）等行为缺陷，需要通过系统化的治理机制来约束。治理是 [[harness-engineering]] 的顶层控制面，与 [[agent-evaluation]] 形成"度量-治理"闭环。

## 治理机制

- **Hook 拦截**：在 Agent 执行的关键节点插入 Hook，拦截和审计 LLM 的决策行为。Hook 可以检测偷懒模式（如跳过非空回复、简化的工具调用）、越权行为（访问未授权的资源）、失忆问题（上下文窗口溢出导致的行为退化）。
- **工具治理**：对 [[skill-system]] 中的工具进行权限分级和调用审计，结合 Function Call 和 MCP 协议实现细粒度的工具访问控制。
- **循环调度管理**：通过 Loopany 等平台，实现多用户、定时、循环的 Agent 任务调度，确保资源分配的公平性和可预期性。
- **面试式准入**：AI-Agent 生产级面试 18 问提供了检查表式的治理评估框架，覆盖安全性、可靠性、可维护性等维度。

## 治理层次

1. **执行前**：权限校验、资源配额检查、策略匹配
2. **执行中**：Hook 实时监测、越权拦截、异常行为检测
3. **执行后**：审计日志、行为评分、反馈归因

## 与相关概念的关系

| 概念 | 关系 |
|------|------|
| [[harness-engineering]] | 治理是 Harness 的顶层控制面 |
| [[skill-system]] | 工具治理的核心对象是 Skill 中的工具调用 |
| [[agent-evaluation]] | 评测数据驱动治理策略的优化 |
| [[multi-agent-orchestration]] | 多 Agent 场景下的治理复杂度指数级上升 |
