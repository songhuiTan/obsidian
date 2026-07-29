---
title: "SubAgent 模式（子代理编排）"
created: 2026-07-21
updated: 2026-07-21
type: concept
tags: [agent-framework, orchestration, harness, methodology]
sources:
  - "技能文档/2026-07-07-深入理解Deep-Agents-Subagents子代理编排机制.md"
  - "技能文档/主流Agent-Harness实现对比-SubAgent与MultiAgent.md"
  - "技能文档/2026-07-18-DeepAgents深度源码解析与工程实践.md"
  - "技能文档/2026-07-06-实测17种Agentic-Loop-Engineering技术.md"
  - "技能文档/2026-06-09-你还在一句句指挥-AI，Anthropic-已经让它自己派一群分身去干活了.md"
confidence: medium
---

# SubAgent 模式

SubAgent 模式是当代 Agent 系统的核心编排范式：将复杂任务分解为多个子任务，由专门的子代理在隔离的上下文中各司其职，最终汇总结果。主 Agent 通过 `task` 工具委派工作，子代理在隔离上下文中完成专业任务。

## 核心价值

| 价值 | 说明 |
|------|------|
| 上下文隔离（Context Quarantine） | 子代理使用独立的上下文窗口，避免大上下文污染 |
| 专门化 | 子代理可使用专属指令、工具集、模型，按职责聚焦 |
| 并行执行 | 多个子代理可同时处理不同任务，提升整体效率 |
| 结构化输出 | 子代理可配置 response_format，确保返回合规结构化数据 |

## SubAgent 的类型

### 通用子代理（General-Purpose）
每个 Deep Agent 自动拥有的同步子代理，可替换或禁用。继承主 Agent 的工具和模型。

### 自定义子代理（字典式）
通过配置定义的专用子代理，关键字段包括：name（唯一标识）、description（面向动作的描述）、system_prompt（专属指令）、tools（可覆盖主 Agent 工具集）、model（可覆盖主 Agent 模型）、response_format（结构化输出 schema）。

### 动态子代理（Beta）
从代码中动态分派子代理，支持循环、分支、并行批处理。

## 与 Multi-Agent 的对比

| 维度 | SubAgent 模式 | Multi-Agent 模式 |
|------|--------------|-----------------|
| 拓扑结构 | 主从架构，主 Agent 协调 | 对等架构，Agent 间直接通信 |
| 上下文隔离 | 强隔离，子代理独立上下文 | 弱隔离，Agent 间共享部分上下文 |
| 控制复杂度 | 低，主 Agent 统一调度 | 高，需通信协议和协商机制 |
| 适用场景 | 任务分解、专业化分工 | 角色扮演、竞争性博弈 |

## 最佳实践

- 描述要具体：面向动作而非模糊描述
- 提示要详尽：分步骤、输出格式清晰
- 工具要精简：最小化工具集，按职责聚焦
- 合理设置 interrupt_on：为关键操作配置 HITL

## 工程实践

[[deep-agents]] 的 SubAgent 机制已深度集成到生产中。实测 17 种 Agentic Loop Engineering 技术表明，SubAgent 模式在处理 20+ 文件变更的大任务时，成功率从 1-20%（单 Agent）提升至 70-95%。

[[hermes-agent]] 的 Skill System 与 SubAgent 模式互补：Skill 提供前置的固化流程，SubAgent 提供后置的动态分解。

## 相关阅读

- [[hitl]] — SubAgent 可独立配置人工介入策略
- [[mcp]] — SubAgent 通过 MCP Server 访问外部工具
