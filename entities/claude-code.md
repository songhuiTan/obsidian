---
title: Claude Code
created: 2026-05-05
updated: 2026-05-05
type: entity
tags: [claude-code, tool, ai-coding, cli]
sources:
  - 知识库/技能文档/OpenClaw-ClaudeCode全栈开发完整指南.md
  - 知识库/技能文档/Claude Code源码泄露分析-2026-03-31.md
  - 知识库/技能文档/Claude Code 开源编译版 free-code.md
confidence: high
---

# Claude Code

## 概述

Claude Code 是 Anthropic 推出的 AI 编程 CLI 工具，具备理解代码库、自主调试、完整工具链等能力。在编程能力上碾压级领先于通用 Agent 框架。

## 与 OpenClaw 的分工协作

典型的全栈开发工作流中两者配合：
```
用户描述需求 → OpenClaw（项目经理）写PRD+技术方案 → 用户确认 → Claude Code（开发者）开发 → 自动化测试 → 交付结果
```

用户只需参与两个节点：确认方案 + 看最终效果，中间全程自动化。

## 实战应用

- **TikTok 爆款分析系统** — 结合 [[opencli]] 和 Playwright 实现全栈开发
- **代码库重构** — 10年54万行PHP系统2周重构为Java
- **全栈网站生成** — 与 [[stitch-2.0]] 配合无限生成网站

## 相关话题

- [[anthropic]] — Claude 模型开发商
- [[openclaw]] — 工作流中的"项目经理"角色
- [[agent-orchestration]] — Agent 编排概念
