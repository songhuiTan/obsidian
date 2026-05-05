---
title: MyHarness
created: 2026-05-05
updated: 2026-05-05
type: entity
tags: [harness, agent-framework, open-source, architecture]
sources:
  - 知识库/技能文档/MyHarness构建思路-从说明书到引擎-2026-04-06.md
  - 知识库/技能文档/gstack+CE组合工作流方案-2026-04-06.md
  - 知识库/技能文档/gstack+CE组合工作流-Harness差距分析与补全方案-2026-04-06.md
confidence: high
---

# MyHarness

## 概述

MyHarness 是一个自定义 Agent 编排引擎，基于对 gstack 和 Compound Engineering（CE）两个开源框架的深度研究构建。定位是"从说明书到引擎"——从静态的组合工作流方案进化为真正的 Agent 执行引擎。

## 构建历程

### 阶段一：研究两个框架
- [[gstack]] — 31+ skill，Sprint 流程（Think→Plan→Build→Review→Test→Ship→Reflect），8 Agent 角色，真实 Chromium 浏览器 QA
- [[compound-engineering]] — 27 Agent（14审查+6研究+3设计+4工作流），核心差异化是 `/ce:compound` 知识沉淀

### 阶段二：组合工作流方案
产出了 `gstack+CE组合工作流方案`，但发现只是"说明书"——没有真正的引擎。

### 阶段三：差距分析
以 OpenHarness 10 子系统架构为基准，发现 5 个核心差距：CLAUDE.md 只是命令列表、无自定义 SKILL.md、无生命周期 Hooks、无 Session 桥接、无自动化 CI。

### 阶段四：引擎构建
代码位置：`/Users/songhuitan/Documents/ai/myharness/`

## 对比其他 Harness

| 维度 | MyHarness | [[deerflow-2.0]] | [[openspace]] |
|------|-----------|-----------------|---------------|
| 定位 | 自定义编排引擎 | Super Agent Harness | 自进化 Skill 引擎 |
| 来源 | 自研 | 字节跳动开源 | HKUDS |
| 核心差异 | OpenHarness 架构对标 | 子智能体并行 | Skill 自动进化 |

## 参考来源
- [[gstack]] — Sprint 流程参考
- [[compound-engineering]] — 知识沉淀参考
- [[agent-orchestration]] — 编排设计
