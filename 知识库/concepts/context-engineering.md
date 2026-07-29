---
title: "Context Engineering（上下文工程）"
created: 2026-07-21
updated: 2026-07-21
type: concept
tags: [methodology, tool, agent-framework, skill-system]
sources:
  - "技能文档/2026-06-11-Symbio进化论08-Token成本优化从PromptCache到语义缓存.md"
  - "技能文档/2026-06-29-Context-Engineering-Kit-改变AI编程workflow开源工具.md"
  - "技能文档/2026-06-30-一次讲清Agent开发最火问题-如何让大模型稳定输出JSON.md"
  - "技能文档/2026-06-04-Headroom上下文压缩工具.md"
  - "技能文档/2026-06-22-Prompt-Canvas-Codex无限画布-图片迭代.md"
  - "技能文档/2026-07-06-阿里SkillWeaver-跳过加载全部工具-Agent-Token直降99%.md"
confidence: medium
---

# Context Engineering

Context Engineering（上下文工程）是指对 AI 编码上下文的工程化管理和优化，目标是让 AI 在有限的上下文窗口内获取最相关的信息，实现极致 Token 高效。

## 核心理念

上下文工程将 AI 编码从"碰运气"推向"可工程化"。核心挑战包括：

- **Token 成本控制**：Prompt Cache、语义缓存等优化技术
- **上下文压缩**：将长内容替换为引用句柄或摘要
- **按需激活**：只在需要时加载相关工具和知识

## 关键工具与技术

### Context Engineering Kit (CEK)
NeoLabHQ 开发的开源工具包，专为 [[claude-code]]、[[codex]]、Cursor 等 AI 编码代理设计。核心插件包括：

- **Reflexion**：自动纠错器，基于 Self-Refine 机制，配合 /memorize 固化经验
- **SDD（Spec-Driven Development）**：完整规范驱动开发流程，集成 architect、reviewer 等子代理
- **SADD（Subagent-Driven Development）**：轻量并行利器，每个子任务用新鲜上下文

### Headroom 上下文压缩
将 Agent 必须接触的长内容降到最少，通过 offload + 引用句柄的方式减少 token 消耗。

### SkillWeaver（阿里）
跳过加载全部工具的机制，Agent Token 直降 99%。只在需要时动态加载相关工具描述。

### Prompt Canvas / Codex 无限画布
支持图片迭代的无限画布工作流，改变传统的线性交互模式。

## 结构化输出工程

让大模型稳定输出 JSON 是上下文工程的一个重要分支，涉及 schema 设计、格式校验、异常处理等多个层面。

## 与 [[hermes-agent]] 的关系

Hermes Agent 的 Skill System 与上下文工程理念一脉相承：通过按需加载技能定义和工具描述，实现最小化上下文占用。
