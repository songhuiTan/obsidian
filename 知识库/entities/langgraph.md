---
title: LangGraph
created: 2026-07-21
updated: 2026-07-21
type: entity
tags: [agent-framework, orchestration, harness]
sources:
  - 技能文档/2026-07-02-为什么选择LangGraph.md
  - 技能文档/LangGraph讲透系列01-为什么需要LangGraph.md
  - 技能文档/LangGraph讲透系列02-项目总览-monorepo核心库和依赖关系.md
  - 技能文档/LangGraph讲透系列03-从最小例子开始-StateGraph是怎么建图的.md
  - 技能文档/LangGraph讲透系列04-状态合并-Annotated与reducer.md
  - 技能文档/2026-07-12-LangGraph讲透系列06-并行分支fan-out与join.md
  - 技能文档/2026-07-14-LangGraph讲透系列08-Checkpoint状态如何持久化与恢复.md
  - 技能文档/LangChain源码解析02-Runnable把一切串起来.md
  - 技能文档/2026-07-12-LangChain源码解析05-Tool如何从函数变成契约.md
  - 技能文档/2026-07-12-LangChain源码解析06-Prompt和Parser守住两端.md
  - 技能文档/2026-07-13-LangChain源码解析07-BaseChatModel如何统一模型调用.md
  - 技能文档/2026-07-15-LangChain源码解析09-create_agent如何编译Agent运行图.md
  - 技能文档/2026-06-04-LangGraph-RAG-Memory-MCP企业级AI助手架构.md
  - 技能文档/主流Agent-Harness实现对比-SubAgent与MultiAgent.md
  - 技能文档/2026-06-16-Agent面试拷打全链路模拟面试.md
  - 技能文档/2026-07-15-LangGraph-OpenAI-SDK-Skills-MCP都不是企业Agent最终答案.md
confidence: medium
---

# LangGraph

## 概述

LangGraph 是 LangChain 出品的有状态图编排框架，主打**显式控制流**。它通过有向图（StateGraph）来定义 Agent 的执行流程，支持状态持久化、并行分支、循环、条件跳转等复杂编排模式。与传统的链式（Chain）调用不同，LangGraph 让开发者明确控制 Agent 的每一步流转和状态变更。

LangGraph 是 LangChain 生态中 Agent 层最核心的编排引擎，底层基于 Runnable 协议构建，与 LangChain 的 Tool、Prompt、Parser、Model 等组件深度集成。

## 关键事实与日期

- **2026-07-02**: 文章《为什么选择LangGraph》分析 LangGraph 相比传统链式调用的优势
- **2026-07-08 至 2026-07-14**: LangGraph讲透系列01-08 系统性剖析框架设计
- **2026-06-04**: LangGraph-RAG-Memory-MCP 企业级AI助手架构设计
- LangGraph 基于 StateGraph 构建，核心概念包括 State、Node、Edge、Checkpoint
- Checkpoint 机制支持状态持久化与恢复，可在任意节点保存和恢复执行状态
- 支持 fan-out/fan-in 并行分支模式
- 使用 Annotated 与 reducer 管理状态合并策略

## 相关文档

- [[deep-agents]] — 同为 LangChain 生态的深度自主 Agent 框架
- [[hermes-agent]] — 开源 OPC 多角色架构 Agent 框架，与 LangGraph 对比
- [[langfuse]] — 可用于 LangGraph 的可观测性平台
- [[claude-code]] — 终端 Agent 编程工具，不同定位的 Agent 实现

## 与其它实体的关系

- **LangChain 生态**: LangGraph 是 LangChain 的核心编排层，与 LangChain 的 Tool、Model、Prompt 组件紧密配合
- **vs DeepAgents**: LangGraph 偏底层图编排，DeepAgents 是上层的高自主 Agent 框架，两者可结合使用
- **vs Hermes Agent**: 不同架构哲学 — LangGraph 显式图控制流 vs Hermes OPC 多角色编排
- **vs Claude Code/Codex**: LangGraph 是通用 Agent 编排框架，Claude Code/Codex 是垂直的编程 Agent

## 来源引用

主要来源：LangGraph讲透系列、LangChain源码解析系列、Agent面试拷打等。详见 sources frontmatter。
