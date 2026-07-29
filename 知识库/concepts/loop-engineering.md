---
title: Loop Engineering
created: 2026-07-21
updated: 2026-07-21
type: concept
tags: [methodology, agent-framework, harness, production]
confidence: medium
sources:
  - Loop-Engineering AI编程生产级闭环 (07-15)
  - 万字图文Loop Engineering (07-05)
  - 实测17种Loop Engineering技术 (07-06)
  - 傻瓜式Loop教程 (07-03)
  - Matt Pocock loop-me (06-25)
  - Claude Code官方Loop教程 (07-06)
  - Loop工程不值钱inspector才值钱 (07-07)
  - Loop Engineering：不再写prompt (06-13)
---

# Loop Engineering

Loop Engineering 是 AI Agent 自动化循环的方法论体系，核心思想是：**不要试图让 LLM 一次性生成完美结果，而是通过精心设计的循环结构，让 Agent 在反馈中逐步逼近目标**。这是 [[harness-engineering]] 在循环控制维度的具体实践。

## 核心循环模型

- **执行-检查-反馈-修正**：Agent 执行一个操作后，自动检查结果质量，将检查结果反馈给自身或外部 Inspector，然后根据反馈修正输出。
- **Inspector 优于 Loop**：循环本身不值钱，真正决定质量的是 Inspector（检查器）的设计。好的 Inspector 能精确指出问题所在，差的 Loop 加上差的 Inspector 只是徒增 token 消耗。
- **闭环终止条件**：循环必须有明确的终止判断——达到质量标准、达到最大轮次、或 timeout。

## 循环模式分类（实测 17 种）

实测总结了 17 种不同的 Loop Engineering 技术，覆盖从简单重试到复杂多 Agent 审核：

1. **简单重试**：失败后自动重试 N 次
2. **分步验证**：每步执行后验证再继续
3. **镜像检查**：用另一个 Agent 审查当前 Agent 的输出
4. **渐进式扩展**：从小范围开始，逐步扩大
5. **测试驱动**：先写测试，再让 Agent 实现
6. **边界试探**：自动探索边界条件

## 与相关概念的关系

| 概念 | 关系 |
|------|------|
| [[harness-engineering]] | Loop 的执行环境，Harness 提供循环的基础设施 |
| [[skill-system]] | 循环中的每一步可以是 Skill 调用 |
| [[agent-evaluation]] | Inspector 本质上是内置的评测机制 |
| [[agent-governance]] | 循环的终止条件和安全护栏属于治理范畴 |

## 工具生态

- **Matt Pocock loop-me**：轻量级 Loop 工具库
- **Claude Code 官方 Loop 教程**：Claude Code 的最佳循环实践
- **Loopany**：多用户定时循环调度平台，见 [[agent-governance]]
