---
title: OpenClaw
created: 2026-07-21
updated: 2026-07-21
type: entity
tags: [agent-framework, harness, tool, skill-system, orchestration, aigc, evaluation]
sources:
  - 技能文档/OpenClaw-ClaudeCode全栈开发完整指南.md
  - 技能文档/OpenClaw爆文分析与养虾实战-2026-03-11.md
  - 技能文档/OpenClaw对普通人工作的帮助-2026-03-11.md
  - 技能文档/2026-03-20-OpenClaw本地可视化看板-Control-Center.md
  - 技能文档/2026-03-16-刚刚，清华团队养出了一只龙虾老师！教育版OpenClaw震撼开源.md
  - 技能文档/2026-06-04-用扣子搭建Agent团队【数字游牧人懒人包】.md
  - 技能文档/DeerFlow 2.0 - 字节Super Agent Harness-2026-03-27.md
  - 技能文档/OpenDeepCrew - AI Agent团队编排服务器-2026-03-27.md
  - 技能文档/2026-03-18-OpenClaw实战：公众号配图全自动化.md
  - 技能文档/Obsidian Direct 技能安装.md
  - 技能文档/2026-03-17-一天94次提交！我用OpenClaw指挥Claude Code，3天搭出一套AI开发团队.md
  - 技能文档/2026-03-16-这两个开源利器，OpenCLI和BB-Browser，让你的OpenClaw瞬间拥有全互联网能力.md
confidence: medium
---

# OpenClaw

## 概述

OpenClaw 是由清华大学团队开源的 Agent 平台/社区，定位为"AI 时代的操作系统"。它以飞书为底座，提供 Skill 技能市场（ClawHub 超 2 万个技能包）、多 Agent 编排、工作流引擎等能力。OpenClaw 的核心理念是让用户通过自然语言指挥 AI Agent 完成复杂任务，从开发编码到内容创作、教育课堂均可覆盖。社区俗称"养龙虾"。

## 关键事实与日期

- ClawHub 技能市场拥有超过 2 万个技能包，覆盖编程、办公、数据分析、浏览器自动化等领域
- 教育版 OpenMAIC 于 2026-03-16 由清华大学教育学院、计算机系联合开源，曾在清华内部跑两年真实课堂
- OpenClaw Control Center 可视化看板于 2026-03-20 由 TianyiDataScience 开源（GitHub 1.1k Star）
- OpenClaw 与 [[Claude Code]] 深度集成构成"项目经理 + 高级开发"协作模式
- 可集成 DeerFlow（字节开源 Super Agent Harness）等外部工作流引擎
- 通过 OpenDeepCrew 实现多 Agent 团队编排
- 支持 Obsidian Direct 技能实现知识沉淀到知识库
- 与扣子（Coze）平台配合搭建多 Agent 团队
- 教育版 OpenMAIC 在国家智慧教育公共服务平台累计访问超 2000 万次
- 2025 年 8 月随清华录取通知书发给每一位新生

## 生态组件

- **教育版 OpenMAIC**：开源多智能体 AI 课堂，AI 老师语音授课、AI 同学举手讨论
- **Control Center**：安全优先、本地优先的可视化监控面板（TypeScript）
- **DeerFlow**：字节 47.3k Star Super Agent Harness，支持 Skills/Tools/Sub-Agents/Sandbox
- **OpenDeepCrew**：AI Coding Agent 团队编排服务器（Server + CLI）
- **OpenCLI / BB-Browser**：赋予 OpenClaw 全互联网操作能力
- **Obsidian Direct**：技能链路，自动整理 AI 对话到 Obsidian 知识库
- **acpx**：底层 agent 会话引擎

## 核心工作模式

OpenClaw 作为"项目经理"角色，负责需求理解、任务拆解、进度管理；[[Claude Code]] 作为"高级开发"负责编码实现。工作流：用户描述需求 -> OpenClaw 写 PRD+技术方案 -> 用户确认 -> Claude Code 开发 -> 自动化测试 -> 交付结果。

## 相关文档

- [[Claude Code]] —— 深度集成的高级编程 Agent
- [[Trellis]] —— 团队级 Agent Harness，与 OpenClaw 互补
- [[OpenSpace]] —— Skill 进化框架，可增强 OpenClaw 技能市场
- [[PilotDeck]] —— 清华同体系的多 Agent 操作系统
- [[WorkBuddy]] —— 生产级 Agent 方法论参考
- [[DeerFlow]] —— 字节开源集成的工作流引擎
- [[Hermes Agent]] —— 同为 Agent 框架，不同设计路径

## 与其他实体的关系

OpenClaw 是清华大学 Agent 生态的核心平台之一，与 [[Trellis]]（团队级 Harness）、[[OpenSpace]]（Skill 进化）、[[PilotDeck]]（多 Agent OS）同属清华/高校系 Agent 基础设施。它与 [[Claude Code]] 形成紧密的"指挥-执行"配合，并通过 DeerFlow、OpenDeepCrew 等组件扩展工作流和多 Agent 编排能力。

## 来源引用

- 技能文档/OpenClaw-ClaudeCode全栈开发完整指南.md
- 技能文档/2026-03-16-刚刚，清华团队养出了一只龙虾老师！教育版OpenClaw震撼开源.md
- 技能文档/2026-03-20-OpenClaw本地可视化看板-Control-Center.md
- 技能文档/DeerFlow 2.0 - 字节Super Agent Harness-2026-03-27.md
- 技能文档/OpenDeepCrew - AI Agent团队编排服务器-2026-03-27.md
