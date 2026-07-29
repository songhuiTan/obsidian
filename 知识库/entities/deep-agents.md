---
title: DeepAgents
created: 2026-07-21
updated: 2026-07-21
type: entity
tags: [agent-framework, harness, skill-system]
sources:
  - 技能文档/2026-07-18-DeepAgents深度源码解析与工程实践.md
  - 技能文档/2026-07-07-深入理解Deep-Agents-Subagents子代理编排机制.md
  - 技能文档/DeepAgents为什么值得学-LangGraph实践者视角.md
  - 技能文档/LangChain-DeepAgents-17个内置中间件详解.md
  - 技能文档/LangGraph-vs-DeepAgents-vs-Hermes-三大Agent框架源码拆解.md
confidence: medium
---

# DeepAgents

## 概述

DeepAgents 是 LangChain 出品的深度自主 Agent 框架，定位为高自主性、长任务执行的 Agent 系统。其核心设计包括**虚拟文件系统（Virtual File System）**、**17 个内置中间件**和 **SubAgent 模式**。与 LangGraph 偏底层的图编排不同，DeepAgents 提供开箱即用的 Agent 能力，包括任务规划、记忆管理、工具调用、子代理委派等。

DeepAgents 的 SubAgent 机制允许主 Agent 动态创建和管理子 Agent，形成层次化的任务执行结构，适用于复杂长任务场景。

## 关键事实与日期

- **2026-07-18**: DeepAgents深度源码解析与工程实践 — 框架核心设计原则与源码分析
- **2026-07-07**: 深入理解Deep-Agents-Subagents — 子代理编排机制详解
- **2026-07-06**: DeepAgents为什么值得学 — 从 LangGraph 实践者视角分析
- **2026-07-06**: LangChain-DeepAgents-17个内置中间件详解 — 中间件体系全览
- **2026-07-06**: LangGraph vs DeepAgents vs Hermes 三大框架对比
- 虚拟文件系统（VFS）：Agent 可读写的抽象文件层，支持任务中间状态持久化
- 17 个内置中间件覆盖：记忆、规划、工具调用、安全检查、日志等维度
- SubAgent 模式：支持 Agent 动态 fork 子任务到子 Agent 执行

## 相关文档

- [[langgraph]] — 同 LangChain 生态的底层图编排框架
- [[hermes-agent]] — 开源 OPC 多角色架构，与 DeepAgents 对比
- [[agentscope]] — 阿里达摩院生产级多智能体框架，不同流派的多 Agent 设计

## 与其它实体的关系

- **LangChain 生态**: DeepAgents 构建在 LangChain 之上，与 LangGraph 互补 — LangGraph 提供底层图执行，DeepAgents 提供高层自主 Agent 能力
- **vs Hermes**: DeepAgents 是 LangChain 官方的深度 Agent 方案，Hermes 是开源社区的多角色架构方案
- **vs AgentScope**: 两者都是多 Agent 框架，但 DeepAgents 更偏自主式 SubAgent，AgentScope 更偏分布式生产级部署
- **中间件体系**: 17 个中间件设计是 DeepAgents 的核心差异化优势

## 来源引用

主要来源：DeepAgents深度源码解析、SubAgent编排机制、三大框架对比等。详见 sources frontmatter。
