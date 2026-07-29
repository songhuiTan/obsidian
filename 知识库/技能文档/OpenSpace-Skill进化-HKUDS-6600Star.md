---
source: 微信公众号
title: "绷不住了，Agent 每次从零烧 token？6.6k Star OpenSpace 把 Skill 变成会进化的资产"
author: 小华（小华同学ai）
date: 2026-07-09
url: https://mp.weixin.qq.com/s/4eN7eGU0bdj_FVU6NGgMAg
tags:
  - OpenSpace
  - Skill 进化
  - Skill 生命周期
  - HKUDS
  - Agent 工程化
---

# 绷不住了，Agent 每次从零烧 token？6.6k Star OpenSpace 把 Skill 变成会进化的资产

> OpenSpace 最值得看的，不是又多了一个 Agent 框架，而是它在解决一个更扎心的问题：**Agent 做完任务后，经验到底有没有留下来？**
>
> 项目地址：https://github.com/HKUDS/OpenSpace（6,660 Star，MIT）

## 核心痛点

| 常见痛点 | 真实表现 | OpenSpace 的方向 |
|----------|----------|------------------|
| token 白烧 | 每次都重新推理、重新试错 | 复用成功流程，减少重复探索 |
| 技能会过期 | API、页面、依赖一变，旧 Skill 静默失效 | 监控质量，触发修复 |
| 经验困在单个 Agent | 一个 Agent 学会了，另一个还要重来 | 通过 Skill 社区共享演化结果 |
| 复杂任务难稳定 | 工具链很长，中间一步错就崩 | 把可靠执行模式沉淀成 Skill |

## 三种 Skill 演化模式

### FIX — 修坏掉的 Skill
当工具、依赖、接口变化导致 Skill 不稳定时，不是简单报错结束，而是尝试定位问题并修复。

### DERIVED — 从旧 Skill 派生更强版本
任务场景需要更细分的能力时，从已有 Skill 演化出更适合的新 Skill，不把所有能力塞进一个大而全的文件。

### CAPTURED — 把成功经验捕获成新 Skill
某次执行里出现可复用的工作流，沉淀出来，下次类似任务不用重新摸索。

**Skill 不再是静态说明书，而是会被真实任务不断打磨的工程资产。**

## Benchmark 数据（GDPVal 评估，骨干 LLM：Qwen 3.5-Plus）

- 50 个专业任务
- 收入提升 4.2 倍
- Phase 2 token 用量约为 Phase 1 的 45.9%
- 50 个 Phase 1 任务中自主进化出 165 项 Skill

> 当任务有重复模式、工具链较长、交付质量需要验证时，Skill 演化才更容易释放价值。

## 接入方式

1. 作为 Agent 的 Skill / MCP 能力接入（Claude Code、Codex、OpenClaw、nanobot 等）
2. 直接把 OpenSpace 当成 AI co-worker 执行编码、搜索、工具调用

## 适用场景判断

| 场景 | 价值 |
|------|------|
| Agent 平台 | Skill 生命周期不能只靠人工维护 |
| 企业自动化 | 成功流程可沉淀成可审计、可复用资产 |
| Coding Agent | 修复工具链、验证输出、复用工作流 |
| 知识/工作流系统 | 经验共享比单次问答更有长期价值 |
| 成本优化 | 降 token 不只靠模型路由，也可以靠少重复试错 |

> **下一阶段的 Agent 工程化，可能不只是拼模型、拼工具，而是拼经验复用能力。**
