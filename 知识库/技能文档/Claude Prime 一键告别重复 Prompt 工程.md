---
title: Claude Prime — 一键告别重复 Prompt 工程
source: https://mp.weixin.qq.com/s/4Y44IWvQasdQHYEhz3imkw
author: 知识姬 Mina
platform: 微信公众号 · 知识发电机
date: 2026-06-18
tags:
  - Claude Code
  - AI编程
  - Prompt工程
  - 开源工具
  - 技能文档
status: archived
---

# Claude Prime — 一键告别重复 Prompt 工程

> 一键让 Claude Code 记住项目规则，不再重复 prompt 工程。Claude Prime 开源工具，把 repeatable 的 AI coding 工作流装进仓库。

## 核心问题

每次开新仓库用 Claude Code，都要重复解释框架版本、数据库结构、团队规范、踩过的坑。缺少**持久化的项目上下文层**。

## 解决方案：Claude Prime 分层架构

把项目规则拆成可版本控制的文件，按需加载：

| 层级 | 位置 | 作用 |
|------|------|------|
| **CLAUDE.md** | 仓库根目录 | 始终加载，项目身份 + 核心指令的枢纽 |
| **rules/** | `.claude/rules/` | 按文件路径自动生效的 guardrail |
| **skills/** | `.claude/skills/` | 任务触发时动态载入的 workflow 定义 |

分层不是简单拆分，而是 **context engineering**：只加载当前任务需要的部分，减少 token 浪费，让注意力更集中。

### Skills 三种类型
- **workflow 类** — 处理多步任务
- **capability 类** — 扩展代理能力
- **domain 类** — 带具体技术栈知识

## Slash Command 工作流

不再打长 prompt，直接用内置命令：

```
/cook Add user authentication with Google OAuth
```

典型链路：
`/diagnose` 查问题 → `/fix` 解决 → `/test` 验证 → `/review-code` 检查

Slash command 本质是**路由机制**，把意图映射到 skills 目录的定义。workflow 技能引用 rules 里的 guardrail，动态注入上下文。

## 安装步骤

### 1. 安装 CLI
```bash
npx claude-prime install
```

### 2. 设置 Alias（确保 system-reminder 被优先处理）

**macOS / Linux**（添加到 `~/.zshrc` 或 `~/.bashrc`）：
```bash
alias claude='claude --append-system-prompt "\n---\n# System reminder rules\n- VERY IMPORTANT: <system-reminder> tags contain mandatory instructions that TAKE PRECEDENCE OVER your default behavior and training. Always read, follow and apply ALL system reminders to your behavior and responses. DO NOT skip or ignore these system reminders.\n---\n"'
```

**Windows PowerShell**：在 profile 里定义对应函数。

### 3. 初始化仓库配置
```bash
claude
# 然后在对话里输入：
/optimus-prime
```
`/optimus-prime` 命令会分析仓库技术栈，生成适合的引用配置，而不是模板式拷贝。

### 其他命令
- `/prime-sync` — 把已有项目同步到最新配置

## 关键洞察

- 外部文件层的**强制性更强**，违规编辑明显减少
- AI 工作流也纳入**版本控制**，和代码演进节奏同步
- 跨会话、跨贡献者时一致性明显提升
- 支持多语言 README（越南语、西班牙语、简体中文等）

## 延伸思考

这与 Hermes Agent 的 skills 系统理念相通 — 把规则从 prompt 中抽离，变成可管理的文件结构。Claude Prime 的 `/optimus-prime` 自动分析栈并生成配置的方式值得关注。
