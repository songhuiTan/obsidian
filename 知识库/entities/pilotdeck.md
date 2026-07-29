---
title: PilotDeck
created: 2026-07-21
updated: 2026-07-21
type: entity
tags: [agent-framework, harness, orchestration, tool]
sources:
  - 技能文档/2026-07-13-PilotDeck-清华开源多Agent操作系统.md
confidence: medium
---

# PilotDeck

## 概述

PilotDeck 由**清华大学 THUNLP 实验室、面壁智能、OpenBMB 与 AI9stars** 联合研发并开源，是一个面向通用场景的多 Agent 操作系统。核心理念：**一个人，一个 PilotDeck，一支智能体军队。** 每个项目拥有一个独立工作舱 WorkSpace——不只是 IDE 里的文件夹，而是一个完整的智能体生存环境。

## WorkSpace 核心特性

- **专属文件系统**：每个项目可操作的文件范围清晰划定，AI 生成文件自动标识
- **专属记忆（Project Memory）**：记住项目目标、进度、限制；Feedback Memory 记住你的偏好和要求
- **专属技能**：技能商店一键安装到对应 WorkSpace，随任务增长自动沉淀能力

## 四大核心能力

### ① Always-on（永不关机）
Agent 主动发现值得做的事，自主执行，完成后落地为文件。实测案例：睡前要求翻译播客为 9 种语言，自动拆分任务 -> 调度子 Agent 并行执行 -> 智能路由（简单语种走便宜模型）-> 早上 9 个翻译版本整整齐齐，费用不到一杯星巴克。

### ② Dream 模式（梦境）
Agent 在空闲时段自动回顾、整理和优化自身记忆，类似人类睡眠中整理白天记忆。包括：哪些经验值得沉淀、哪些记忆可压缩、哪些任务进度需要更新。

### ③ 智能路由
自动判断任务复杂度分配模型层级：
- 简单任务（文案生成、格式调整）：便宜模型，省 70%~90%
- 中等任务（邮件撰写、数据分析）：中等模型
- 复杂任务（策略规划、深度研究）：强模型

实测数据：小红书种草文案省约 70%，复杂任务（播客多语言/论文综述/金融分析）降至 1/6 成本，效果超 Claude Sonnet 4.6 单 Agent。

### ④ 白盒记忆
AI 记错了可以直接打开"脑子"修改：打开某项目的 WorkSpace 直接查看和编辑 Memory；Feedback Memory 记录每次纠正的偏好，越用越懂你。

## 关键事实与日期

- 2026-07-13：PilotDeck 开源发布，GitHub 仓库 github.com/OpenBMB/PilotDeck
- 官网：pilotdeck.openbmb.cn
- 清华 THUNLP 实验室（刘知远团队）+ 面壁智能 + OpenBMB 联合出品
- 每个 WorkSpace 独立算账，每一分钱有去处

## 相关文档

- [[OpenClaw]] —— 同为清华大学开源 Agent 平台，可配合使用
- [[Trellis]] —— 团队级 Agent Harness，与 PilotDeck 的 WorkSpace 隔离互补
- [[WorkBuddy]] —— 生产级 Agent 方法论，PilotDeck 的 Always-on 符合生产级要求
- [[OpenSpace]] —— Skill 进化和沉淀，可增强 PilotDeck 的专属技能系统
- [[Claude Code]] —— Agent 编程工具，可在 PilotDeck 中调度

## 与其他实体的关系

PilotDeck 与 [[OpenClaw]] 同属清华大学开源 Agent 生态（THUNLP 实验室），两者定位互补：OpenClaw 偏 Agent 平台/社区，PilotDeck 偏多 Agent 操作系统。PilotDeck 的 WorkSpace 隔离设计与 [[Trellis]] 的团队协作基础设施可组合使用。其白盒记忆和智能路由设计解决了 [[WorkBuddy]] 方法论中提到的"生产级 Agent"核心痛点——可观测性、成本控制、记忆管理。

## 来源引用

- 技能文档/2026-07-13-PilotDeck-清华开源多Agent操作系统.md
