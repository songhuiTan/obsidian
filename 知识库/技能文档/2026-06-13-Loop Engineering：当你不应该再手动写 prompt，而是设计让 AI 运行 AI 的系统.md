---
title: "Loop Engineering：当你不应该再手动写 prompt，而是设计让 AI 运行 AI 的系统"
author: AI开源速递
source_url: "https://mp.weixin.qq.com/s/Ay1Tt35fOFVGHlSphk6-tw"
date: "2026-06-13"
tags: [Loop Engineering, AI Agent, Claude Code, Codex, 自动调度, 生产模式]
---

# Loop Engineering：当你不应该再手动写 prompt，而是设计让 AI 运行 AI 的系统

过去两年你用 AI Coding Agent 的方式，大概率是这样的：打开终端，敲一段 prompt → 等代码生成 → 读一遍 → 发现不对 → 再敲一段 prompt → 再等 → 再改。

你拿着鼠标，AI 是你的打字员。你一局一局地指挥它，它一局一局地执行。

这个模式正在被一个叫 Loop Engineering 的概念重新定义。Boris Cherny（Anthropic Claude Code 负责人）的原话是：

「我不再给 Claude 写 prompt 了。我有 loops 在运行，这些 loops 负责给 Claude 发指令、决定下一步做什么。我的工作变成了写 loops。」

Peter Steinberger（OpenClaw 创始人）说得更直接：「你早就不该手动给 coding agent 写 prompt 了。你应该设计 loops，让 loops 替你给 agent 发指令。」

Addy Osmani 6月7号发了一篇长文，把这套理念系统化，取名叫 Loop Engineering。一位叫 Cobus Greyling 的工程师紧接着做了一个完整的参考实现（110⭐ 但内容密度非常高），包含 7 个生产级模式、3 个 CLI 工具、一个交互式展示站。

我在周末仔细读了一遍，然后试着把工具跑了一遍。

● ● ●

## Loop Engineering 到底是什么

定义很简单：**Loop Engineering 是「用系统替代你自己，作为驱动 AI agent 的那个人」**。

传统工作流是：你 → 写 prompt → AI → 返回结果 → 你 → 再写 prompt。

Loop Engineering 的工作流是：你 → 设计一个循环系统 → 这个系统自动发现任务 → 分配给子 agent → 执行 → 验证 → 更新状态 → 决定下一步 → 循环。

注意差别——你不再参与每一次交互。你设计的是规则和边界，具体的调度和执行由循环系统自动完成。

下面是 Loop Engineering 的完整工作流架构图，从调度触发到最终提交的完整流程：

![Loop Engineering 完整工作流架构图](../assets/2026-06-13-Loop%20Engineering/img_001.png)

● ● ●

## 五个基础构件 + 记忆

一个能无人值守运行的生产级 loop，需要六个东西：

调度（Automations）：按时间或事件触发的发现和分诊机制。比如每天早8点自动扫描 GitHub Issues。在 Codex 里是 Automations tab，在 Claude Code 里是 Scheduled tasks + /loop + /goal

工作树（Worktrees）：让多个 agent 并行工作时互不踩踏的隔离机制。Claude Code 用 git worktree，Codex 内置了每线程一个 worktree

技能（Skills）：把项目知识固化下来，agent 不用每次猜。格式就是 SKILL.md（OpenClaw 协议）

插件与连接器（Plugins & MCP）：把 agent 接进你已经在用的工具——GitHub、Linear、Slack。底层是 MCP 协议

子 agent（Sub-agents）：写代码和审代码的人分开。一个子 agent 负责实现，另一个负责验证和测试

记忆/状态（Memory/State）：这是最容易被低估但最重要的一块。Agent 每次运行后都会「失忆」，所以状态必须放在磁盘上，不能放在上下文中。通常是一个 STATE.md 文件，或者 Linear board，记录「做了什么」「下一步做什么」「哪里卡住了」

Addy 的原话：「听起来太简单了不值一提。但它是每一个长期运行 agent 都依赖的同一个技巧——模型在两次运行之间会忘光所有东西，记忆必须在磁盘上，不能在上下文中。」

● ● ●

## 七个生产模式

Loop Engineering 的参考实现提供了 7 个可以直接用的模式：

Daily Triage（每日分诊）：每天运行一次，扫描 Issues/PRs/通知，生成报告。L1 阶段只报告不操作，L2 阶段开始自动修复低风险问题。token 消耗低

PR Babysitter（PR 保姆）：每个新 PR 触发，自动做代码审查、跑测试、检查 lint。token 消耗高（因为要读完整 diff）

CI Sweeper（CI 清扫）：每次 CI 失败后自动分析日志、定位根因、提交修复。token 消耗极高

Dependency Sweeper（依赖清扫）：每 6 小时扫描依赖更新，自动提交低风险的补丁升级

Changelog Drafter（更新日志起草）：按 tag 或每日自动生成 changelog 草案。token 消耗低

Post-Merge Cleanup（合后清理）：PR 合并后自动清理分支、更新文档、关闭关联 issue

Issue Triage（Issue 分诊）：自动给新 Issue 打标签、分配负责人、判断优先级

每个模式都有明确的「安全等级」：

· L1：只报告，不改动任何代码

· L2：可以修低风险问题，但需要人工审核

· L3：完全无人值守，自动提交和合并

这个分阶段设计很聪明——第一天你只是看看 loop 在干什么，观察它的判断力。等信任建立后，再逐步放开权限。

● ● ●

## 实操体验：loop-audit 工具

我跑了一下 loop-audit 工具检查当前项目的 loop 就绪度：

```
npx @cobusgreyling/loop-audit . --suggest
```

输出是一个完整的诊断报告，检查了 16 项指标：有没有 STATE.md、有没有 LOOP.md、有没有 skills 目录、有没有安全检查、有没有 token 预算文档、有没有运行日志……

我的项目得分是 10/100，L0 级别（还没就绪）。

但最有价值的是每条失败项后面都跟着具体建议：「复制 minimal-loop 的 STATE.md.example 到 STATE.md」「添加 loop-triage skill」「在 LOOP.md 中记录 human gate 策略」——这不是一个只会说「你不行」的检测工具，而是一个会告诉你「下一步怎么做」的教练。

● ● ●

## 这个理念和 Hermes Agent 的对照

写到这里我意识到一件事——我在 Hermes Agent 里用的很多模式，其实已经是 Loop Engineering 的雏形了。

Hermes 的 cronjob 系统（定时任务、脚本轮询、无 agent 模式）对应 Loop 的调度层。skills 对应技能层。delegate_task 对应子 agent 模式。notify_on_complete 对应 loop 的异步通知。

差别在于，Hermes 里这些是「功能点」，需要用户自己拼装成循环。而 Loop Engineering 把它们提升为一等设计模式——有命名、有协议、有状态管理、有故障处理。

● ● ●

## Caveats：不是没有代价的

Loop Engineering 的理念很诱人，但参考实现自己也列了一堆警告：

Token 消耗可能爆炸。子 agent + 长时间运行的 loop，token 消耗量是普通对话的 10-50 倍。PR Babysitter 模式被标记为「很高」token 成本，CI Sweeper 更是「极高」。如果后端跑的是 Claude Opus，一上午可能烧掉几十美元

验证责任还在你身上。无人值守的 loop 犯的错也是无人值守的。如果没有完善的测试和验证体系，loop 可能会把 bug 合并进主分支

理解债务增长更快。AI 写的代码越来越多，但人读代码的速度基本没变。除非你定期阅读 loop 提交的内容，否则项目会逐渐变成一个只有 AI 能理解的黑箱

两个工程师跑同一个 loop 可能得到完全相反的结果。Loop 不负责判断力，你负责

● ● ●

## Loop 参考实现的结构

这个参考实现的代码组织方式也值得提一下：

```
loop-engineering/
  patterns/         7 个生产模式，每个有自己的 markdown 文档
  starters/        可直接克隆的脚手架（minimal-loop, pr-babysitter 等）
  tools/            三个 CLI 工具
    loop-audit/     loop 就绪度检测
    loop-init/      脚手架生成器
    loop-cost/      token 消耗估算
  docs/             设计文档（失败模式、反模式、安全检查、多 loop 协调）
  examples/         各平台的具体实现（Grok、Claude Code、Codex）
  assets/           视觉素材和架构图
```

每个 starter 目录里都包含 LOOP.md（循环配置）、STATE.md（状态文件）、skills/（技能）、safety.md（安全策略）。

● ● ●

## 适合谁关注这个理念

已经在用 Claude Code / Codex / Cursor 做日常编码、想放大 AI 产出的人

对 Agent 架构设计感兴趣的开发者——Loop Engineering 和 Harness Engineering 是同一套思想的两个层次

管理多个代码仓库或 GitHub 项目的团队——自动分诊和 PR 审查可以节省大量人力

项目已经有一套 CI/CD 体系，想在上面叠加 AI 层的人

● ● ●

## 入门步骤

想试 Loop Engineering，最轻量的切入点是 Daily Triage 模式：

```
npx @cobusgreyling/loop-init . --pattern daily-triage --tool grok
```

这会自动生成 LOOP.md、STATE.md、skills/loop-triage 等文件。

然后跑一次审计看就绪度：

```
npx @cobusgreyling/loop-audit . --suggest
```

第一次运行建议用 L1 模式（只报告不操作），观察一周，等对 loop 的输出建立信任后再升级到 L2。

用 loop-cost 估算 token 消耗：

```
npx @cobusgreyling/loop-cost --pattern daily-triage --level L1
```

● ● ●

## 一点看法

Loop Engineering 这个理念让我觉得有意思的点在于——它不是某个大厂发布的战略，而是一个从一线工程师实践中自下而上涌现的模式。Boris Cherny 说「我的工作就是写 loops」，Peter Steinberger 说「你不该再手动写 prompt」，Addy Osmani 把它写成文章，Cobus Greyling 把它做成参考实现。一周之内，一个概念从 tweet 变成了有代码、有工具、有模式的完整体系。

这种演化速度在传统软件开发领域是看不到的。

但它也确实处在一个早期阶段：7 个模式、3 个 CLI 工具、110 个 star。这是理念验证的规模，不是生产就绪的规模。如果你现在就想在核心项目上跑 L3 无人值守 loop，风险还很大。

但那个方向是对的——从「人手动调教 AI」到「人设计让 AI 调教 AI 的系统」。这件事正在发生。

## 参考资源

Addy Osmani 原文：https://addyosmani.com/blog/loop-engineering/

参考实现：https://github.com/cobusgreyling/loop-engineering

Substack 文章：https://cobusgreyling.substack.com/p/loop-engineering

交互式展示：https://cobusgreyling.github.io/loop-engineering/
