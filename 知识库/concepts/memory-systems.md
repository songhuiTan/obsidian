---
title: "Agent 记忆系统（Memory Systems）"
created: 2026-07-21
updated: 2026-07-21
type: concept
tags: [memory, agent-framework, knowledge-base, archive]
sources:
  - "技能文档/2026-06-01-NexSandglass-沙漏记忆系统-本地优先AI-Agent记忆引擎.md"
  - "技能文档/2026-06-16-AgentMemory共享工程记忆层.md"
  - "技能文档/2026-06-30-OpenHands源码解析一-控制面数据面拆分.md"
  - "技能文档/2026-07-14-LangGraph讲透系列08-Checkpoint状态如何持久化与恢复.md"
  - "技能文档/2026-07-07-AI-Agent生产级面试18问.md"
  - "技能文档/2026-07-13-DeepTutor-港大开源AI个性化辅导私教-25KStar.md"
  - "技能文档/2026-07-06-HereVault-把AI对话变成可检索的知识资产.md"
confidence: medium
---

# Agent 记忆系统

Agent 记忆系统是解决 AI Agent 跨会话上下文保持的核心方案。在实际生产中，Agent 面临"上下文不够"和"关键信息遗忘"的双重挑战：新会话不知道上次改到哪，多个 Agent 之间的经验教训无法复用。

## 记忆系统的分层架构

从多个实践项目（[[langgraph]]、NexSandglass、AgentMemory）中可以归纳出通用的记忆分层模型：

### L0 — 缓冲层（Buffer）
短期会话上下文，如 LangGraph 的 thread-scoped checkpoint。保存当前会话的运行状态和节点执行进度。核心数据结构包括 channel_values（业务状态）、channel_versions（调度时钟）、versions_seen（节点消费进度）。

### L1 — 持久层（Persistence）
跨会话的持久化存储。NexSandglass 通过 SQLite 实现零依赖的本地持久化，支持 FTS5 全文搜索。AgentMemory 则通过 Session - Observation - Memory - Lesson 四层模型将任务痕迹沉淀为可复用的工程记忆。

### L2 — 检索层（Retrieval）
多种检索策略的组合。NexSandglass 的四路并发搜索（FTS5、IDX、TF-IDF、Shadow Sand）提供了低延迟的记忆检索能力，影子沙（Shadow Sand）仅需 0.7ms 即可完成最优匹配。

### L3 — 推理层（Reasoning）
记忆的高阶应用，包括偏移率追踪（Drift Velocity）、情绪熵分析、知识图谱构建（织线三元组）等。

## 关键设计模式

**Checkpoint 机制**：LangGraph 的 checkpoint 不仅保存状态值，还保存通道时钟和节点消费进度，支持精确恢复执行现场。

**共享工程记忆**：AgentMemory 通过 Hooks 捕获 Agent 工作过程中的关键事件，形成 Observation，再提炼为 Memory 和 Lesson，实现跨 Agent / 团队的共享记忆。

**记忆注入优化**：NexSandglass 实现每轮仅约 60 token 的极简注入，LLM 按需调用搜索，大幅降低 token 消耗。

## 相关技术

- [[hermes-agent]] 的 Memory Provider 机制
- [[claude-code]] 的 CLAUDE.md 经验固化
- HereVault 将 AI 对话转化为可检索的知识资产
- DeepTutor 的个性化辅导记忆
