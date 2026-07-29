---
title: Hermes Agent
created: 2026-07-21
updated: 2026-07-21
type: entity
tags: [hermes, agent-framework, orchestration, harness]
sources:
  - 技能文档/2026-07-17-Hermes五角色模型3.0-一人公司OPC架构设计.md
  - 技能文档/2026-06-22-Hermes-多角色架构OPC模式-一人公司（抖音视频）.md
  - 技能文档/2026-06-06-autoresearch到auto-alpha-Agent自主研究范式实战.md
  - 技能文档/LangGraph-vs-DeepAgents-vs-Hermes-三大Agent框架源码拆解.md
  - 技能文档/2026-06-17-1次安装5大Agent入驻-OhMyHermes.md
  - 技能文档/hermes-llm-wiki实战/2026-05-05-Hermes满配指南-7步.md
  - 技能文档/hermes-llm-wiki实战/2026-05-05-Hermes资源库-飞书归档.md
  - 技能文档/hermes-llm-wiki实战/Hermes 这个技能我一直没碰，跑完一遍后悔没早试.md
confidence: medium
---

# Hermes Agent

## 概述

Hermes Agent 是由 Nous Research 开发的开源 Agent 框架，核心设计为 **OPC（Orchestrator-Planner-Coder）多角色架构**，后演进为五角色模型。框架将 Agent 拆分为**协调员（Coordinator）**、**研究员（Researcher）**、**写作者（Writer）**、**构建者（Builder）** 等角色，每个角色专注特定职能，通过协调层统一调度。

Hermes 的设计理念是"一人公司"——让单个用户通过多角色 Agent 协作完成复杂知识工作。它强调 Agent 之间的角色分工和协作，而非单一 Agent 执行所有任务。

## 关键事实与日期

- **2026-07-17**: 五角色模型3.0发布，进一步细化角色分工
- **2026-06-22**: OPC 多角色架构详解（抖音视频）
- **2026-06-06**: autoresearch 到 auto-alpha 实战 — Agent 自主研究范式
- 五角色模型：协调员、研究员、写作者、构建者、审核员
- OPC 架构：Orchestrator（协调）- Planner（规划）- Coder（编码）三元组
- 开源项目，社区驱动发展
- 强调多角色协作而非单 Agent 全能

## 相关文档

- [[deep-agents]] — LangChain 出品的深度自主 Agent，对比架构设计
- [[langgraph]] — LangChain 图编排框架，不同编排哲学
- [[agentscope]] — 阿里达摩院生产级多智能体框架
- [[langfuse]] — 可观测性平台，可用于 Hermes 的监控评测

## 与其它实体的关系

- **vs LangGraph/DeepAgents**: Hermes 采用角色分工编排，与 LangChain 的图编排或自主 Agent 模式形成对比
- **vs Claude Code/Codex**: Hermes 是通用 Agent 框架，非垂直编程 Agent
- **OPC 架构**: 是 Hermes 区别于其他框架的核心设计
- **开源社区**: Nous Research 维护，活跃的社区贡献

## 来源引用

主要来源：Hermes五角色模型3.0、OPC多角色架构、autoresearch实战等。详见 sources frontmatter。
