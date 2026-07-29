---
title: "LangGraph vs DeepAgents vs Hermes — 三大 Agent 框架路线深度对比"
created: 2026-07-21
updated: 2026-07-21
type: comparison
tags: [comparison, agent-framework, orchestration, harness]
sources:
  - 技能文档/LangGraph-vs-DeepAgents-vs-Hermes-三大Agent框架源码拆解.md
  - 技能文档/LangGraph讲透系列01-为什么需要LangGraph.md
  - 技能文档/2026-07-18-DeepAgents深度源码解析与工程实践.md
  - 技能文档/2026-07-17-Hermes五角色模型3.0-一人公司OPC架构设计.md
  - 技能文档/2026-07-02-为什么选择LangGraph.md
confidence: medium
---

# LangGraph vs DeepAgents vs Hermes

## 三大 Agent 框架路线总览

| 对比维度 | LangGraph | DeepAgents | Hermes Agent |
|----------|-----------|------------|--------------|
| 维护方 | LangChain | LangChain | Nous Research |
| 核心范式 | 图状态机（StateGraph） | 深度自主 + 虚拟文件系统（VFS） | OPC 多角色架构 |
| 编程模型 | 显式 DAG + 节点/边/状态 | SubAgent 层次委派 + 中间件洋葱模型 | 协调员-规划员-编码员等角色分工 |
| 状态管理 | Checkpoint 机制，Annotated reducer | VFS 持久化 + 中间件上下文 | 多角色协作上下文 |
| 并行能力 | fan-out/fan-in 内置 | SubAgent 异步 Fork | 角色并行执行 |
| 控制流显式程度 | 极高（完全显式图） | 中（中间件链 + 自主调度） | 中（角色编排协议） |
| 企业级特性 | Checkpoint + LangSmith | 17 个内置中间件 | 开源社区扩展 |
| 学习曲线 | 中高（需理解图论概念） | 中（中间件体系较大） | 中低（角色模型直观） |
| 适用场景 | 需要精确控制的管道和工作流 | 长任务、高自主性场景 | 知识工作协作、一人公司模式 |

## 架构哲学对比

**LangGraph** 走的是"显式控制流"路线。所有 Agent 行为都映射为有向图上的节点和边，状态变更由 reducer 明确定义。这种方法让开发者对 Agent 每一步都有绝对控制权，适合需要精确编排的企业级工作流。

**DeepAgents** 走的是"深度自主"路线。通过 17 个内置中间件和 VFS，Agent 被赋予更大的决策自由。SubAgent 模式允许动态 fork 子任务，适合需要长时间自主执行的任务。与 LangGraph 互补而非替代——LangGraph 提供底层图执行，DeepAgents 提供高层自主能力。

**Hermes Agent** 走的是"多角色协作"路线。受到 OPC（协调-规划-编码）架构启发，框架将 Agent 能力拆分为不同角色（协调员、研究员、写作者、构建者、审核员），通过角色间协作完成复杂知识工作。设计目标是"一人公司"——让单个用户通过多 Agent 协作完成全流程。

## 选型建议

- 需要**精确控制每一步执行**的场景（如金融合规、医疗诊断）→ [[langgraph]]
- 需要**长期自主执行复杂任务**的场景（如代码库大规模重构、数据分析报告）→ [[deep-agents]]
- 需要**多角色协作完成知识生产**的场景（如研究报告撰写、内容创作、一人公司运营）→ [[hermes-agent]]
- 大型企业可考虑 **LangGraph + DeepAgents 组合使用**：LangGraph 编排底层管道，DeepAgents 负责上层自主执行
- 社区驱动的小团队或个人开发者推荐从 **Hermes** 入手，门槛最低

## 综合评价

三个框架代表了 Agent 编排的三种不同哲学：**图编排**（LangGraph）、**自主 Agent**（DeepAgents）、**角色协作**（Hermes）。它们不是简单的竞争关系，而是互补关系——理解了这三种路线，才能在具体场景中做出正确的技术选型。
