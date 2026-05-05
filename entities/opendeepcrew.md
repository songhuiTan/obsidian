---
title: OpenDeepCrew
created: 2026-05-05
updated: 2026-05-05
type: entity
tags: [agent-framework, open-source, orchestration, tool]
sources:
  - 知识库/技能文档/OpenDeepCrew - AI Agent团队编排服务器-2026-03-27.md
confidence: high
---

# OpenDeepCrew

## 概述

OpenDeepCrew 是一个 AI Coding Agent 团队编排服务器（Server + CLI）。Alpha 阶段，v0.0.23，2026-03-26 发布，通过 [acpx](https://github.com/openclaw/acpx) 驱动 Agent 生命周期。

GitHub: `@opendeepcrew/opendeepcrew`

## 核心功能

- **工作区隔离** — 每个工作区独立配置 agent、MCP servers、hooks、skills
- **插件市场** — 从本地路径或 git 仓库加载 commands/agents/skills/hooks
- **会话管理** — 创建、监控、取消 agent 会话，实时流式日志
- **多 agent 初始化** — 一键为 Claude Code、Cursor、Kiro 生成工作区配置
- **飞书机器人** — 聊天驱动 agent 交互，支持图片/文件/视频，多机器人池
- **团队模式** — spawn agent 团队，@mention 路由，按成员读取 inbox
- **Web 控制台** — 浏览市场、管理工作区、监控会话
- **优雅重启** — 配置变更触发零停机重启（exit code 120 协议）

## 架构

```
Web UI (React) ↔ OpenDeepCrew (Express API) ↔ acpx (sessions)
                        ↕
                 Marketplace (plugins/atoms)
```

## 相关实体

- [[myharness]] — 同赛道自定义 Harness
- [[deerflow-2.0]] — 字节跳动同赛道产品
- [[openclaw]] — 底层依赖 acpx 驱动
- [[agent-orchestration]] — Agent 编排概念
