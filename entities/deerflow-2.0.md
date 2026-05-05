---
title: DeerFlow 2.0
created: 2026-05-05
updated: 2026-05-05
type: entity
tags: [harness, agent-framework, open-source, company]
sources:
  - 知识库/技能文档/DeerFlow 2.0 - 字节Super Agent Harness-2026-03-27.md
confidence: high
---

# DeerFlow 2.0

## 概述

DeerFlow 2.0 是字节跳动开源的 Super Agent Harness 框架，GitHub 47.3k ⭐。位于 Agent Framework 之上的更高层架构，管理长期任务的执行基础设施。

项目地址：https://github.com/bytedance/deer-flow

## 什么是 Agent Harness

Agent Harness 是包裹在 AI 模型周围的基础设施，专门用于管理长期任务。提供预设 Prompt、工具调用标准化、生命周期钩子、开箱即用的规划/文件系统/子智能体管理能力。

> "Agent 是强壮却野性难驯的骏马，Harness 不提供动力，却能牢牢牵住方向、稳住步伐。" — Philipp Schmid

## AI 工程三次跃迁

1. **2023-2024 提示工程** — 教人类怎么跟 AI 说话
2. **2025 上下文工程** — 精算给 AI 看什么信息
3. **2026 Harness** — 为模型构建可执行、可信赖、可长期运转的数字世界

## 核心能力

- **Skills & Tools** — 内置研究、报告生成、幻灯片等，Tools 可通过 MCP Server 扩展
- **Sub-Agents** — 复杂任务自动拆解，主智能体按需动态拉起子智能体，可并行执行
- **多 Agent 团队** — 每个子智能体有独立上下文、工具、终止条件

## 相关实体

- [[myharness]] — 自定义 Harness，参考了 DeerFlow 的设计
- [[openspace]] — 另一方向的自进化 Skill 引擎
- [[opendeepcrew]] — Agent 团队编排
- [[agent-orchestration]] — 编排设计概念
