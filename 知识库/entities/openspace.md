---
title: OpenSpace
created: 2026-07-21
updated: 2026-07-21
type: entity
tags: [agent-framework, skill-system, methodology, harness]
sources:
  - 技能文档/OpenSpace-Skill进化-HKUDS-6600Star.md
  - 技能文档/OpenSpace核心机制分析与MyHarness优化方案-2026-04-06.md
confidence: medium
---

# OpenSpace

## 概述

OpenSpace（HKUDS，GitHub 6,660 Star，MIT）不是一个传统的 Agent Harness，而是一个**自进化 Skill 引擎**。其核心命题是：Skill 不应该是静态的 Markdown 文件，而应该在每次使用中学习、适应、进化。OpenSpace 定位在 Agent 工程化中"经验复用"这一关键缺口——让 Agent 完成复杂任务后，经验能真正沉淀下来，而不是每次从零烧 token。

## 三种 Skill 进化模式

- **FIX（修复）** —— 技能没起作用时分析原因，定位问题并就地修复。当工具、依赖、接口变化导致 Skill 不稳定时触发。
- **DERIVED（派生）** —— 任务场景需要更细分的能力时，从已有 Skill 组合出更适合的新 Skill，不把所有能力塞进一个大而全的文件。
- **CAPTURED（捕获）** —— 某次执行里出现可复用的工作流后沉淀为新 Skill，下次类似任务不用重新摸索。

## 三个进化触发器

1. **执行后分析** —— 每次执行后自动分析技能是否有效
2. **工具退化检测** —— 某工具成功率下降时触发
3. **周期性指标检查** —— 每 5 次执行检查一次整体质量

## 技能质量指标体系

- `applied_rate` = total_applied / total_selections
- `completion_rate` = total_completions / total_applied
- `effective_rate` = total_completions / total_selections
- `fallback_rate` = total_fallbacks / total_selections

## 关键事实与日期

- 2026-04-06：OpenSpace 核心机制分析文档归档
- 2026-07-09：OpenSpace 详细拆解文章发布（公众号"小华同学ai"）
- Benchmark 数据（GDPVal 评估，骨干 LLM：Qwen 3.5-Plus）：50 个专业任务，收入提升 4.2 倍，Phase 2 token 用量约为 Phase 1 的 45.9%，50 个 Phase 1 任务中自主进化出 165 项 Skill
- 技能谱系（Lineage）记录：parent_skill_ids、generation、change_summary、content_diff、content_snapshot
- 混合搜索：候选技能超过 10 个时先用 BM25 粗排，再用 Embedding 精排，最后用 LLM 选出最合适者
- 可接入 Claude Code、Codex、[[OpenClaw]]、nanobot 等平台

## 相关文档

- [[OpenClaw]] —— 可通过 OpenSpace 增强其 ClawHub 技能市场
- [[Trellis]] —— Spec 团队规范体系可与 OpenSpace 的 Skill 进化结合
- [[Claude Code]] —— OpenSpace 可接入的主要编码 Agent 平台
- [[WorkBuddy]] —— 其第六阶段"能沉淀能力"与 OpenSpace 的 Skill 进化理念一致
- [[Hermes Agent]] —— 同为 Agent 基础设施，不同设计取向

## 与其他实体的关系

OpenSpace 关注的是 [[Trellis]] 和 [[OpenClaw]] 都未深度处理的技能生命周期管理。Trellis 把 Spec 做成 git 版本化团队资产，OpenSpace 则更进一步让 Skill 在每次使用中自动进化。与 [[WorkBuddy]] 六阶段中的"能沉淀能力"阶段理念相通——高频有效经验需要沉淀为 Skill。OpenSpace 可以嵌入任何 Agent 平台作为 Skill 进化的底层引擎。

## 来源引用

- 技能文档/OpenSpace-Skill进化-HKUDS-6600Star.md
- 技能文档/OpenSpace核心机制分析与MyHarness优化方案-2026-04-06.md
