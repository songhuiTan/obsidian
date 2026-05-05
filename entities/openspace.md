---
title: OpenSpace
created: 2026-05-05
updated: 2026-05-05
type: entity
tags: [skill, open-source, architecture, research]
sources:
  - 知识库/技能文档/OpenSpace核心机制分析与MyHarness优化方案-2026-04-06.md
confidence: high
---

# OpenSpace

## 概述

OpenSpace（HKUDS）不是一个 Harness，而是一个**自进化 Skill 引擎**。其核心命题：Skill 不应该是静态的 Markdown 文件，而应该在每次使用中学习、适应、进化。

## 三种进化模式

| 模式 | 触发条件 | 效果 |
|------|---------|------|
| FIX（修复） | 技能没起作用 | 分析原因，就地修复 |
| DERIVED（派生） | 从已有技能组合 | 组合出新的更强技能 |
| CAPTURED（捕获） | 发现新模式 | 创建全新的技能 |

## 三个进化触发器

1. **执行后分析** — 每次执行后自动分析技能是否有效
2. **工具退化检测** — 某工具成功率下降时触发修复
3. **周期性指标检查** — 每 5 次执行检查一次整体质量

## 技能谱系（Lineage）

```
skill_v1 → skill_v2 (FIXED) → skill_v3 (FIXED)
skill_A → skill_C (DERIVED, from A + B)
```

## 与 MyHarness 的关系

本知识库中研究了 OpenSpace 作为 [[myharness]] 的设计参考。其自进化理念被部分采纳到 MyHarness 的优化方案中。

## 相关概念

- [[skill-development]] — Skill 开发方法
- [[myharness]] — 采纳了 OpenSpace 理念的 Harness
- [[agent-orchestration]] — Agent 编排
