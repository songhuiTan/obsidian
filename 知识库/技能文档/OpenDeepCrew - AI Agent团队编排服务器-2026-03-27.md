---
title: OpenDeepCrew - AI Coding Agent 团队编排服务器
source: https://www.npmjs.com/package/@opendeepcrew/opendeepcrew
author: OpenDeepCrew (dev@opendeepcrew.com)
archived: 2026-03-27
tags: [AI-Agent, 编排, coding-agent, 飞书, MCP, marketplace]
---

# OpenDeepCrew - AI Coding Agent 团队编排服务器

> Alpha 阶段，API 和配置格式可能会变。v0.0.23，2026-03-26 发布。

## 简介

Server + CLI，用于编排 AI coding agent 团队。管理工作区、会话、插件市场，可选飞书机器人集成。底层通过 [acpx](https://github.com/openclaw/acpx) 驱动 agent 生命周期。

## 核心功能

- **工作区隔离**：每个工作区独立配置 agent、MCP servers、hooks、skills
- **插件市场**：从本地路径或 git 仓库加载 commands/agents/skills/hooks
- **会话管理**：创建、监控、取消 agent 会话，实时流式日志
- **多 agent 初始化**：一键为 Claude Code、Cursor、Kiro 生成工作区配置
- **飞书机器人**：聊天驱动 agent 交互，支持图片/文件/视频，多机器人池
- **团队模式**：spawn agent 团队，@mention 路由，按成员读取 inbox
- **Web 控制台**：浏览市场、管理工作区、监控会话、查看日志、编辑设置
- **优雅重启**：配置变更触发零停机重启（exit code 120 协议）

## 架构

```
┌──────────┐     ┌──────────────┐     ┌─────────┐
│ Web UI   │────▸│ opendeepcrew │────▸│  acpx   │
│ (React)  │◂────│ (Express API)│◂────│(sessions)│
└──────────┘     └──────┬───────┘     └────┬────┘
                        │                   │
                 ┌──────▼───────┐    ┌──────▼─────┐
                 │ Marketplace  │    │ Claude,    │
                 │(plugins/atoms)│    │ Kiro, ...  │
                 └──────────────┘    └────────────┘
```

## 安装

```bash
npm install -g @opendeepcrew/opendeepcrew@latest
npm install -g acpx@latest  # agent 会话引擎
opendeepcrew server         # 启动，默认端口 4000
```

首次启动自动运行初始化向导（端口、市场源、启用的 agent 类型）。

## CLI 命令速查

```bash
# 服务
opendeepcrew server [--port 8080]

# Marketplace
opendeepcrew marketplace add --type <skill|command|hook|agent> <source>
opendeepcrew marketplace show
opendeepcrew marketplace edit          # IDE 打开
opendeepcrew marketplace plugin list|create|delete
opendeepcrew marketplace atom list [--type skill]
opendeepcrew marketplace history [-n 50]
opendeepcrew marketplace rollback <commit>
opendeepcrew marketplace sync

# Config
opendeepcrew config show|get|set <key> <value>

# Workspace
opendeepcrew workspace list
opendeepcrew workspace open <name>
opendeepcrew workspace update <name> --permission-mode <approve-all|approve-reads>
```

## API 端点

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/health | 健康检查 |
| GET | /api/marketplace/plugins | 插件列表 |
| POST | /api/marketplace/refresh | 重新加载市场 |
| GET/POST/DELETE | /api/workspaces | 工作区管理 |
| POST | /api/workspaces/:name/reinit | 重新初始化 |
| GET | /api/sessions | 活跃会话列表 |
| GET | /api/sessions/:id/logs | 流式日志 |
| POST | /api/sessions/:id/cancel | 取消 prompt |
| DELETE | /api/sessions/:id | 关闭会话 |
| GET/PUT | /api/settings | 配置管理 |
| POST | /api/settings/restart | 优雅重启 |

## 飞书集成

- 长连接模式（无需公网域名）
- 支持文本/图片/视频/音频/文件/飞书文档
- 自定义菜单触发工作流（event_key: `add_requirement`）
- 多机器人池（TEAM_MULTIBOT_ENABLED）
- 媒体自动下载，UUID 命名，2 分钟 TTL 自动清理
- 出站自动识别图片/文件路径并上传飞书

### 权限要点
- 主机器人需完整 im + contact + cardkit + helpdesk 权限
- 池中机器人仅需 im:message + im:message:send_as_bot

## 技术栈

- Node.js >= 20，TypeScript
- Express 5，Commander 14
- Pino 日志，Husky + lint-staged
- 前端：React 19（client 子目录）
- 飞书：@larksuiteoapi/node-sdk

## 相关项目

| 项目 | 说明 |
|------|------|
| [acpx](https://github.com/openclaw/acpx) | Agent 会话引擎 |
| [marketplace](https://github.com/open-deep-crew/marketplace) | Atom 市场模板 |
| [acpx-teams](https://pypi.org/project/acpx-teams/) | Agent 团队协调 MCP server |

## 许可

MIT
