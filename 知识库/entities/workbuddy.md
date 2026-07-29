---
title: WorkBuddy
created: 2026-07-21
updated: 2026-07-21
type: entity
tags: [methodology, enterprise, production, orchestration, harness]
sources:
  - 技能文档/2026-07-15-拆完WorkBuddy-我看到了生产级Agent的完整形态.md
  - 技能文档/2026-07-12-生产级Agent全景-架构Harness工程组织与人才-叶小钗.md
  - 技能文档/WorkBuddy绩效自评8个Prompt模板.md
confidence: medium
---

# WorkBuddy

## 概述

WorkBuddy 由叶小钗系统阐述，是一套**生产级 Agent 平台和方法论**。它回答的核心是：为什么 90% 的 Agent 最终都进不了生产环境？核心主张是"找场景 > 选框架"——Agent 产品起点不是选模型、选框架，而是先用场景价值公式（场景价值 = 任务价值 x 可执行程度 x 可验证程度 / 失败风险）找到高价值闭环。

## 关键事实与日期

- 2026-07-12：叶小钗发布《生产级 Agent 全景：架构、Harness 工程、组织与人才》
- 2026-07-15：叶小钗发布《拆完 WorkBuddy，我看到了生产级 Agent 的完整形态》
- 提出"现阶段不存在通用 Agent，先找真实场景里的高价值闭环"

## 生产级 Agent 六阶段演进路径

1. **能聊** —— Chat UI + 模型调用 + 流式输出 + 历史消息（解决体验入口）
2. **能用工具** —— Tool calling，按业务场景渐进，不要一上来接很多工具
3. **能做任务** —— 引入 task、step、status、artifact（用户看到完整任务在推进）
4. **能跑长任务** —— Runtime、checkpoint、compression、pause/resume、retry policy、cost tracking
5. **能被治理** —— 安全权限、Token 成本面板、操作日志、回滚能力
6. **能沉淀能力** —— 高频有效经验沉淀为 Skill，包括工程经验和 Agent 参数

## 三类典型 Agent 场景

| 类型 | 代表场景 | 核心关键 |
|------|---------|---------|
| 生产力工具型 | AI Coding、AI 文档、AI 研究助手 | 工程能力 + 用户体验 |
| 业务流程型 | 数字员工、CRM 自动化、工单处理 | 模型推理 + 业务 KnowHow + 系统集成 |
| 多模态 AIGC | 图片/视频/海报生成 | 效果评估复杂、依赖模型组合 |

## 生产级架构：双视角

### 用户视角四层
1. **入口层** —— Chat、模板、文件、事件、集成入口
2. **执行层** —— 核心是可观测性：用户能看到计划、步骤、工具调用、中间结果
3. **产出层** —— 产出必须是完整的、一致的、可保存可修改的
4. **治理层** —— 可控、可信、可复盘、可改进

### 工程视角五大系统
1. **大脑系统**（Model + Agent Loop）—— 循环：观察 -> 理解 -> 计划 -> 选工具 -> 执行 -> 观察结果 -> 更新状态
2. **行动系统**（Tools + Runtime）—— 工具不在多，选错成本高
3. **工作台系统**（State + Context + Memory）—— 生产级 Agent 设计的核心难点
4. **经验系统**（Prompt + Skill + Workflow）—— 薄 Harness，厚 Skill
5. **控制系统**（HITL + Guardrails + Eval）—— 高风险动作前让人确认

## SaaS -> Workflow -> Agent 演进链

SaaS 记录业务状态 -> Workflow 执行固定流程 -> Agent 理解上下文并泛化行动 -> 数字员工承担稳定工作。Workflow 和 Agent 不存在谁替代谁，核心根据步骤明确度与失败代价来分配自由度。

## 相关文档

- [[Trellis]] —— 其团队级 Harness 设计实现了 WorkBuddy 的多项生产级要求
- [[OpenClaw]] —— 作为 Agent 平台，是 WorkBuddy 方法论的落地载体
- [[OpenSpace]] —— Skill 进化能力与第六阶段"能沉淀能力"理念一致
- [[PilotDeck]] —— Always-on 设计和智能路由实现了生产级要求
- [[Claude Code]] —— 生产力工具型场景的代表
- [[Hermes Agent]] —— 可作为生产级 Harness 的参考实现

## 与其他实体的关系

WorkBuddy 是一套评价和指导所有 Agent 平台/框架的方法论。[[Trellis]] 的团队协作设计和跨会话记忆管理可视为对 WorkBuddy 第四阶段"能跑长任务"和第五阶段"能被治理"的实践。[[OpenSpace]] 的 Skill 进化直接对应第六阶段"能沉淀能力"。WorkBuddy 的"场景 > 框架"理念适用于评估所有 Agent 项目。

## 来源引用

- 技能文档/2026-07-15-拆完WorkBuddy-我看到了生产级Agent的完整形态.md
- 技能文档/2026-07-12-生产级Agent全景-架构Harness工程组织与人才-叶小钗.md
- 技能文档/WorkBuddy绩效自评8个Prompt模板.md
