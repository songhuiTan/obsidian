---
title: "Loopany — 多用户定时Agent循环调度平台（BYOA + 零执行架构）"
source: "superdesigndev / Loopany"
source_url: "https://github.com/superdesigndev/loopany-platform"
date: "2026-07-18"
tags: [AI, Agent, 开源, 调度系统, BYOA, TanStack, pnpm, TypeScript, Loop]
---

## 项目定位

**Loopany** 是一个多用户定时 Agent 循环调度平台。核心模型：用你自己的机器的 coding agent 跑定时任务，服务器的职责只到调度、存储、认证和通知——**从不运行 LLM，从不执行用户代码**（zero-exec invariant）。

一句话：**Agent 界的 cron + 团队面板。**

> "Describe a recurring task once. Loopany runs it on a schedule with your own machine's coding agent, and surfaces every result on a shared dashboard and your team's notification channel."

## 架构拆解

### 两层部署拓扑

```
┌──────────────────────────────────┐
│        Loopany Server            │  ← 你部署 or loopany.ai
│  TanStack Start (React + Vite)   │
│  Scheduler (croner)              │
│  Machine Gateway + Agent API     │
│  Better Auth + Push Notify       │
│  Drizzle ORM → Postgres/pglite   │
│  R2 blob store (artifacts)       │
│  Zero LLM · Zero code-exec       │
└────────────────┬─────────────────┘
                 │ HTTP poll (long/short)
                 ▼
┌──────────────────────────────────┐
│  @crewlet/loopany Daemon         │  ← 你控制的机器
│  npx 安装，一个二进制两个角色     │
│  Poll 调度 → spawn claude-code   │
│  workflow gate (纯JS前置)         │
│  artifacts · MCP bridge          │
│  skill 自动安装                   │
└──────────────────────────────────┘
```

**零执行不变量：** 服务器只调度/存储/认证/通知，不运行任何 LLM 调用，不执行任何用户代码。执行完全在用户的机器上通过 daemon + local coding agent 完成。

### 核心数据模型

- **Loop**（循环）：一次定时任务的完整定义。分为 open loop（持续监控/摘要，无限运行）和 closed loop（有目标，达成后自动标记完成）。
- **Machine**（机器）：绑定到用户的一台本地机器，通过 daemon 与服务器通信。
- **Run**（运行）：一次 tick 的完整执行记录。有三种角色：
  - `exec` — 定时调度运行，产生用户通知
  - `evolve` — 自我改进运行，审查历史记录优化 loop 本身
  - `edit` — 用户请求的修改运行
- **Artifact**（产物）：运行时同步回服务器的文件（报告、看板卡片、日历等）。

### 关键设计决策

1. **HTTP Poll 而非 WebSocket**：IDLE 时用长轮询（~20s hold，近零延迟），有 run in flight 时切换到短轮询（~3s 心跳）。不需要常驻连接，daemon 可以重启。

2. **三种 Run 角色分离**：exec（面向用户）、evolve（自我改进）、edit（用户修改）。只有 exec 产生用户通知。evolve 会分析历史运行、优化提示词、把机械步骤收编为脚本——让 loop 越跑越聪明、越跑越便宜。

3. **Deterministic Pre-stage（workflow gate）**：可选的纯 JS 前置工作流。对于传感器读取→摘要这类任务，workflow 直接返回结果，无需 spawn LLM。只有 workflow `agent()` 升级调用时才跑 coding agent。这是零 LLM 路径的工程实现。

4. **Prompt 即代码**：所有 prompt 文案放在 `packages/server/src/skill/` 下，按受众拆分为 `SKILL.md`（公共可安装 skill）、`references/`（运行时引用）、`run/`（exec/evolve/edit 各自的任务 body）、`templates/`（模板）。系统 prompt 为空字符串——所有指令通过第一个 user turn 传递，确保旧版 daemon 也能兼容。

5. **CLI 的 TOON 协议**：所有 `/api/machine/cli` 返回使用自定义的 `toon.ts` 序列化格式（类似 gh-axi 的 TOON 格式），保证 CLIs 端的一致性。

## 核心特性详解

| 特性 | 描述 |
|------|------|
| **定时 Agent 循环** | cron 或一次性；open loop（持续监控）和 closed loop（有完成目标，达成后自动禁用） |
| **BYOA 执行** | 你的机器 + 你的 agent（claude-code/codex），服务器不碰你的代码和密钥 |
| **自我改进 (Evolve)** | 周期性的 evolve run 审查历史、优化任务说明、提炼状态、完善 dashboard |
| **确定性前置 (Workflow)** | 可选的纯函数工作流，机械步骤前置处理，失败后回退到 agent |
| **通知渠道** | Telegram、飞书等多渠道推送，团队共享仪表盘 |
| **产物同步** | loop 目录 → dashboard 渲染；支持 generative UI（报告、看板卡片、日历） |
| **Skill 自动安装** | daemon 自动在 claude-code/codex 上安装 loopany skill（每次 `up`/`new` 时） |
| **模板** | React Doctor、Market Research、Follow-up Tracker、Docs Sweep、Housekeeper、Dependency Triage、Error Sweep |
| **自托管** | 一个进程启动，嵌入 pglite 本地零依赖跑，生产用 Postgres + S3 对象存储 |

## 技术栈总结

### 服务器端（@loopany/server）

| 类别 | 组件 | 用途 |
|------|------|------|
| 框架 | TanStack Start (React 19 + Vite 8 + Nitro 3) | UI + 服务端函数 + 构建 |
| 路由 | TanStack React Router | 客户端/服务端路由 |
| 数据库 | Drizzle ORM + Postgres（pglite 嵌入 / Supabase） | schema + 迁移 + 查询 |
| 认证 | Better Auth (+ GitHub OAuth) | 登录/授权 |
| 调度 | croner | 进程内定时器引擎 |
| 存储 | AWS S3 SDK (R2) | 产物 blob 存储 |
| UI | Base UI + Tailwind CSS 4 + Recharts + CodeMirror | 组件库 + 样式 + 图表 + 代码编辑器 |
| 通知 | 自定义通知模块 | Telegram/飞书推送 |
| 测试 | Vitest + jsdom | 测试框架 |

### 守护进程端（@crewlet/loopany）

| 类别 | 组件 | 用途 |
|------|------|------|
| 运行时 | Node.js >= 22 | 执行环境 |
| 二进制入口 | `loopany` (dist/cli.js) | 一个二进制两个角色（daemon / 运行回调） |
| Agent 兼容 | Claude Code / Codex | spwan 并驱动 |
| Workflow 引擎 | 自定义 runner (JS AST) | 前置确定性执行 |
| MCP 桥接 | mcp-bridge.mjs | 在 agent 运行时暴露 Loopany 工具 |
| 文件监听 | chokidar | 监听 loop 目录变化 |
| 日志 | pino + pino-pretty | 结构化日志 |

## 同类项目对比

| 维度 | Loopany | Cron + Script DIY | LangGraph Cloud | TaskWeaver (MS) |
|------|---------|-------------------|-----------------|-----------------|
| 执行位置 | **你的机器**（BYOA） | 你的机器 | 云端 | 你的机器 |
| 服务器 LLM | **零 LLM** | N/A | 有（LLM 编排） | 有（LLM 编排） |
| Agent 绑定 | **vendor-neutral** | 无限制 | LangChain 生态 | 插件体系 |
| 团队面板 | ✅ 共享 dashboard | ❌ 无 | ✅ 有 | ❌ 无 |
| 自我改进 | ✅ Evolve 机制 | ❌ | ❌ | ❌ |
| 产物 UI 渲染 | ✅ Generative UI | ❌ | ❌ | ❌ |
| 通知渠道 | ✅ 多平台 | ❌ 需自写 | ✅ 有 | ❌ |
| 自托管复杂度 | 低（一个进程 + pglite） | — | 中 | 高 |
| 安装方式 | `npx @crewlet/loopany` | crontab | API 调用 | 本地部署 |

## 当前状态与待办

| 状态 | 方面 |
|------|------|
| ✅ 已实现 | 调度引擎、daemon poll/spawn、exec/evolve/edit 三角色、workflow gate、产物同步、通知、CLI TOON 协议、skill 自动安装 |
| ✅ 已实现 | 认证（GitHub OAuth + open 模式）、模板系统、Dashboard UI |
| ✅ 已实现 | 自托管（pglite + Docker + Fly.io 部署配置） |
| ⚠️ 早期阶段 | 项目明确标注 early-stage，daemon 权限较高（执行 coding agent 持你的凭据） |
| 🔜 计划中 | AGENTS.md 提及的能力版本管理、回滚、UI 完善、运行状态聚合 |
| 🔜 计划中 | 内容助手的长期选题去重、跨任务素材复用 |

## 文档体系亮点

| 文档 | 行数 | 用途 |
|------|------|------|
| **README.md** | ~350 行 | 面向用户的定位 + 快速开始 + 部署说明 |
| **AGENTS.md** (→ CLAUDE.md) | ~60K 字符 | **极致详尽的架构手册**：布局、命令、核心模型、prompt 体系、CLI 协议、每个 gate 的设计决策、已知 bug 修复记录、迁移批次说明 |
| **CONTRIBUTING.md** | ~100 行 | 贡献指南（迁移、发布、PR 流程） |
| **packages/server/src/skill/** | 7 个文件 | 所有 prompt 文案按受众拆分，公共 skill 可安装 |
| **.env.example** | ~200 行 | 所有环境变量 + 注释说明 |

AGENTS.md 是文档体系的亮点——它不只是架构手册，更像一个**活的决策日志**（含 batch 1-7 的演进说明、已修复的 bug F1-F5、每个协议的约束条件），是 AI 协作式开发（Claude Code driven）的典范产物。

## 架构价值评估

### 优势

1. **真正的 zero-exec invariant** — 服务器不跑 LLM 不执行代码，用户数据不出机器。这是目前见过的 Agent 调度系统中对安全和隐私最干净的方案。对比 LangGraph Cloud 需要在云端运行 LLM，对比 TaskWeaver 需要在本地运行完整框架。

2. **BYOA 的 vendor-neutral 设计** — 不绑定到特定 Agent 供应商。当前支持 claude-code 和 codex，通过 `LOOPANY_AGENT` 配置即可扩展。Agent 选择权在用户。

3. **三种 Run 角色的分层设计** — exec/evolve/edit 分离解决了 Agent loop 的核心问题：不是一直做同样的事，而是随时间自我优化。Evolve 机制是最独特的创新——市面上几乎没有其他平台能让定时 Agent 任务自己改写自己。

4. **Workflow gate** — 可选的确定性前置步骤，让机械工作（传感器读取、简单汇总）直接完成，无需 spawn LLM。这是在成本和响应速度上的工程智慧。

5. **文档成熟度极高** — AGENTS.md 是业界最好的 Agent 项目架构文档之一（和 Hermes 的文档站同级）。适合作为 TypeScript Agent 项目的架构参考。

### 可借鉴的点

1. **Prompt 即代码 + 通过 user turn 传递**：系统 prompt 为空，所有指令通过第一个 user turn 传递。这样旧版 daemon（不支持新版 prompt 格式）也能运行——回退兼容做得干净。

2. **CLI 协议用统一 TOON 格式**：`toon.ts` 确保所有 `/api/machine/cli` 返回格式一致，daemon 和 CLI 消费者只解析一种格式。

3. **Skill 自动安装机制**：daemon 在 `up`/`new` 时自动在 claude-code 和 codex 上安装 loopany skill。这确保 Agent 总是知道如何与 Loopany 交互。

### 注意事项

1. **早期阶段**：项目明确标注 early-stage，daemon 权限较高（执行 coding agent 持你的凭据）。安全加固持续进行中。
2. **生态依赖**：当前只支持 claude-code 和 codex，扩展其他 agent 需要手动配置。
3. **单进程限制**：服务器只能跑一个进程（in-process scheduler 的约束），两个进程共享同一个 DB 会重复触发运行。

## 与 Hermes Agent 的整合分析

| 维度 | Loopany | Hermes Agent | 整合潜力 |
|------|---------|-------------|---------|
| 定时调度 | ✅ 内建 croner | ❌ 无内建调度（有 cronjob 但为对外投递） | **高** — Hermes 可以接 Loopany 作为定时 Agent 任务的调度层 |
| Agent 执行 | 本地 claude-code/codex | Hermes 子代理 + gateway | 互补 — Hermes 管执行细节，Loopany 管调度和团队面板 |
| 团队协作 | ✅ 多用户 + dashboard + 通知 | ❌ 单人工具 | Loopany 补 Hermes 的团队协作缺口 |
| 产物管理 | ✅ 同步 + generative UI | ❌ 输出到 chat | 可借鉴 Loopany 的 artifact 同步模式 |
| 零执行 | ✅ 服务器不跑 LLM | ❌ 服务器就是 Agent | 理念对立，但不冲突 |

Loopany 解决的是"定时让 Agent 干活 + 团队看见结果"，Hermes 解决的是"Agent 怎么干活"。两者的结合点是：用 Loopany 调度 Hermes 子代理作为执行后端（替代 claude-code）。

不过 Hermes 作为 Python/TS 工具链，对接 Loopany 需要实现一个 Hermes-centric 的 daemon 或 bridge。

## 相关资源

- **Loopany 官网**：https://loopany.ai
- **GitHub 仓库**：https://github.com/superdesigndev/loopany-platform
- **npm 包**：https://www.npmjs.com/package/@crewlet/loopany
- **许可证**：MIT（所有包均为 MIT）
- **作者/组织**：Superdesign（© 2026）
- **文档站**：GitHub README + AGENTS.md 为核心

## 归档日志

- 2026-07-18 归档
