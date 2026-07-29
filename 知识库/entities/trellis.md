---
title: Trellis
created: 2026-07-21
updated: 2026-07-21
type: entity
tags: [agent-framework, harness, skill-system, orchestration, methodology]
sources:
  - 技能文档/Trellis-团队级Agent-Harness框架-拆解vs-Superpowers.md
  - 技能文档/2026-07-14-Trellis定制工作流-trellis-meta大项目五层防御.md
confidence: medium
---

# Trellis

## 概述

Trellis（GitHub 12k Star）是一个**团队级 Agent Harness 框架**，具备内置 LLM Wiki。它定位在团队协作基础设施层面，解决跨会话、跨任务、跨团队的"项目失忆症"与规范不统一问题。Trellis 不依赖模型记住规范，而是通过"文件即记忆"的设计让规范以 git 版本化文件形式存在仓库中。

## 三层架构

```
Agent Harness（执行骨架）——管理 workflow 状态、hook、skill、子代理调度
内置 LLM Wiki（知识库）——Spec（团队规范）+ Task（任务知识）+ Journal（会话记忆），以文件形式存入仓库
团队层（协作层）——文件全部 git 版本化，团队共享，适配 16 个 AI 编程平台
```

## 关键事实与日期

- 2026-07-07：Trellis 团队级 Agent Harness 框架拆解文章发布
- 2026-07-14：trellis-meta 定制工作流发布，提出大项目五层防御体系
- JSONL manifest 精确点名任务所需文件，按固定顺序注入
- 跨会话状态管理：task 状态标记（planning -> in_progress -> 归档）
- 按开发者区分的持久 journal，可无限期跨会话续接
- Spec 库 git 版本化共享，`update-spec` 沉淀为全团队规则
- `.trellis/` 目录跨 16 个平台复用，`trellis init` 仅生成适配文件

## trellis-meta 五层防御体系（大项目长任务）

1. **意图澄清** —— 第一性原理 + 苏格拉底提问，产出 prd.md
2. **路线锁定** —— route-lock-guardian 产出 route-lock.md / non-goals-and-killed-routes.md / permission-matrix.md
3. **方案挑刺** —— grill-me 苏格拉底反问 + 自我苏格拉底七问
4. **任务契约** —— "完整薄片"原则：每个子任务有完整输入/处理/输出/接入点/测试/证据
5. **测试先行** —— 用测试定义"做对了"，苏格拉底提问自拷测试方案

## 与 Superpowers 对比

| 维度 | Trellis | Superpowers |
|------|---------|-------------|
| 核心定位 | 团队级 Agent Harness + 内置 LLM Wiki | 面向单会话的软件工程方法论 |
| 核心问题 | 跨会话/跨团队"项目失忆症" | 单任务上下文膨胀 |
| 记忆持续性 | 跨会话状态机 | 单任务隔离 |
| 团队协作 | Spec git 版本化共享 | 个人技能覆盖 |
| 跨工具适配 | 16 个平台自动生成适配层 | 每换环境单独安装 |

## 相关文档

- [[OpenClaw]] —— 同为高校系 Agent 平台，可互补使用
- [[OpenSpace]] —— Skill 进化框架，Trellis 的 Spec 体系可与之结合
- [[Claude Code]] —— Trellis 适配的主要编码 Agent 平台之一
- [[WorkBuddy]] —— 生产级 Agent 方法论，与 Trellis 的工程化思路互补
- [[Hermes Agent]] —— 同类 Agent Harness，不同设计路径

## 与其他实体的关系

Trellis 不追求取代 [[Claude Code]] 或 [[OpenClaw]]，而是作为团队级基础设施，解决这些底层 Agent 工具在团队协作场景中的空白。它与 [[Superpowers]] 不是竞品，而是解决不同维度的问题——Superpowers 管单个开发流程，Trellis 管团队协作基础设施。trellis-meta 的定制工作流可视为 [[WorkBuddy]] 六阶段方法论在 Trellis 生态中的具体实现。

## 来源引用

- 技能文档/Trellis-团队级Agent-Harness框架-拆解vs-Superpowers.md
- 技能文档/2026-07-14-Trellis定制工作流-trellis-meta大项目五层防御.md
