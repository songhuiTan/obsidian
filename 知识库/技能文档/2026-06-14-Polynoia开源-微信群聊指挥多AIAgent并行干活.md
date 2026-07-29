---
title: "Polynoia开源：微信群聊指挥多AI Agent并行干活！5+1状态机+12+Artifacts一码三端深度解析"
source: "如此才是"
source_url: "https://mp.weixin.qq.com/s/nDf89WBq3yug850oXh-f9Q"
date: "2026-06-14"
tags: [AI, Agent, 多Agent协作, 开源, 状态机, FastAPI, React]
---

## 概述

Polynoia 是一个开源的 IM 风格多 Agent 协作平台。核心创新在于：**Orchestrator（协调器）本身就是一个可插拔的 Agent**，会自动解析意图、生成任务 DAG、并行分发给群成员、验证产出、处理冲突并执行 git 合并。整个过程对用户透明，像在微信群里 @ 几个同事干活一样自然。

项目基于 **FastAPI + React** 构建，同一套 Vite 构建产物可同时运行在 Web、桌面（Tauri 2）和移动端（Capacitor 6），真正做到一码三端。

## 核心亮点

| 维度 | 典型 AI coding 工具 | **Polynoia** |
|------|-------------------|--------------|
| 心智模型 | 一个助手、一条线程 | **一个团队**，支持 1:1 和群聊 |
| 并行能力 | 顺序轮次 | **Orchestrator 并行分发** + burst lanes |
| 输出形态 | 文本+代码块 | **12+种富结构化 Artifacts**，可预览可编辑 |
| 多引擎支持 | 锁定单一厂商 | **统一 Adapter 层**（Claude Code / Codex / OpenCode） |
| 工作合并 | 手动 copy-paste | **每 Agent 独立 git worktree** + 引导式冲突解决 |
| 运行平台 | 仅桌面浏览器 | **Web + Tauri 桌面 + Capacitor 移动** 一码三端 |

## 架构与核心设计

### 1. Orchestrator 5+1 状态机

运行一个 ~700 LOC 的状态机（`apps/server/polynoia/orchestrator/runtime.py`），严格 5+1 阶段：

1. **INTENT_PARSE**：流式输出规划 + JSON 任务清单（DAG 结构）
2. **DISPATCH + AWAIT_BARRIER**：DAG runner 并行调度子任务，使用 `FIRST_COMPLETED` 收集；UI 显示为并行 burst lanes
3. **AGGREGATE**：收集输出、冲突检测、Orchestrator 合并结果
4. **EMIT_PREVIEW**：有 diff 时向聊天注入富预览卡片
5. **MERGE**（P1.2 auto 模式）：逐分支执行 `git merge --no-ff`，产出结果卡 + 新 main SHA

**失败处理**：`RunningTask` 记录 `attempts/max_attempts=2`，失败自动重排 pending 重试；`CancelledError` 不重试。冲突目前引导人工 side-by-side 解决（LLM 自动解决计划 P2）。

### 2. 统一 Adapter 层 + 自定义 Agent

一套协议（**PAP - Polynoia Adapter Protocol**，11 种 `AdapterEvent` Pydantic 判别联合）适配三种引擎：
- **Claude Code**：Claude Agent SDK，强推理+长上下文
- **OpenCode**：ACP v1（JSON-RPC/NDJSON），开源本地优先标准
- **Codex**：`codex` app-server 流式

**关键设计**：Adapter（引擎）与 Contact（persona + tools + tags）完全解耦。一个引擎可派生多个角色。支持从一句话描述创建自定义 Agent，工具粒度开关（read_file/edit_file/run_shell/network/call_agent 等），自动派生 capability tags。

### 3. Inline Artifacts（12+种富消息部件）

消息不是纯文本，而是 `MessagePart[]` 通过注册表分派渲染。一条回复可混排：`text` · `reasoning` · `tasks` · `diff` · `web` · `metrics` · `sql` · `schema` · `logs` · `api` · `swatches` · `copy` · `file` · `image` · `ask-form` · `typing`。

支持 Markdown WYSIWYG、Marp 幻灯片、可编辑 Office 文档、CodeMirror 6 代码、实时 Web 预览（iframe + CSP sandbox）、Git 历史等。

### 4. Workspace IDE + 冲突闭环

绑定 workspace 后右侧 PreviewPane 变身迷你 IDE：文件树 + CodeMirror 6（VS Code 键位、minimap、search/replace），`Ctrl+S` → PUT 保存 → 自动 commit main，交互式 PTY 终端，GitHub 风格 commit 历史 + side-by-side diff。

**冲突是一等公民**：并行 Agent 在独立分支上工作必然冲突 → 聊天中出现冲突卡 → 打开引导式并排解决面板 → 解决后自动 commit 回 main，全程用自然语言解释。

### 5. 流式与跨平台

采用 **Vercel AI SDK 6 `UIMessageChunk`** 协议通过 WebSocket 推送，前端刷新或断线重连后能精确断点续传。

**一码三端**：
- **Web**：完整体验
- **Desktop（Tauri 2）**：包裹 web 构建，支持切换自定义本地/LAN/远程后端
- **Mobile（Capacitor 6）**：WeChat 风格 4-tab（Chats · Agents · Projects · Me）

### 6. 沙箱与安全

**双模式沙箱**：per-conv（独立 git + credentials）或 workspace-shared（共享 git + 分支/工作树）。**安全策略**：绝不存储 API Key，工具白名单（Orchestrator 为空），网络允许列表仅 LLM endpoint + npm + pypi。

## 关键技术原理

- **整体分层**：React 18 + Vite + Radix + shadcn/ui + Tailwind 4 + Zustand + CodeMirror 6 + Vercel AI SDK 6（客户端）；FastAPI + asyncio + Pydantic v2 + SQLAlchemy 2.0 async + aiosqlite/asyncpg（服务端）
- **Context 5 层 Assembler**：L1 identity 硬 2k token（永不削减）→ L2 项目 briefs 软 3k → L3 跨会话 ledger 软 15k → L4 当前历史软 35k（cursor 分页）→ L5 当前 turn 硬 5k。自研 CJK-aware token 估算（中文×1.5）、per-message cap、2-pass 预算强制
- **消息协议三层**：Adapter↔Server（PAP）→ Server↔Client（AI SDK 6 UIMessageChunk）→ Client→Server（REST + WS 命令）
- **21 个 ADR** 记录了所有重大取舍

## 关键洞察

- **Orchestrator 必须是一个 Agent**（而非硬编码逻辑），这一设计让 Orchestrator 可替换、可定制，用户可以不用它或替换它的 persona
- 同 Adapter 多 Contact = 独立人格（独立 ledger），底层引擎共享但 persona 和上下文完全隔离
- Claude Code 的 `system_prompt` 使用 append 模式（保留内置 prompt），避免破坏 Claude Code 原生能力
- 整个项目用 AI 作为一等协作者构建（CLAUDE.md 规范 + Conventional Commits）

## 战略分析

### 与现有工具链的对照

Polynoia 与本地的 Hermes Agent 体系在理念上互补而非冲突：

| 维度 | Polynoia | Hermes Agent（当前体系） |
|------|----------|------------------------|
| 定位 | 多 Agent 协作平台 | 个人 Agent 运营中枢 |
| 交互范式 | IM 聊天（群聊/1:1） | CLI + WeChat 对话 |
| 多 Agent 编排 | Orchestrator 状态机 + DAG | delegate_task 子代理 |
| Agent 引擎 | Claude Code / Codex / OpenCode | Hermes 原生（多 provider） |
| 记忆系统 | 5 层 Context Assembler | TencentDB 四层记忆 + NexSandglass |
| 输出形态 | 12+ 富 Artifacts（预览/编辑） | 纯文本 + 媒体文件 |
| Git 集成 | 一等公民（worktree + merge） | 无原生集成 |
| 跨平台 | Web + Desktop + Mobile 一码三端 | WeChat + CLI |
| 开源 | ✅ MIT | ✅ 开源 |
| 自定义 Agent | 一句话创建 + 工具粒度开关 | 通过 skill 系统扩展 |

### 差距与借鉴点

**值得借鉴的设计：**
1. **Orchestrator 状态机**（5+1 阶段）—— 当前 `delegate_task` 的编排逻辑位于代码层面，没有显式的状态机模型。Polynoia 的 `INTENT_PARSE → DISPATCH → AGGREGATE → EMIT_PREVIEW → MERGE` 阶段划分清晰，可直接映射到 Hermes 的执行流程。
2. **MessagePart 注册表模式** —— 当前 Hermes 的回复是纯文本 + 附件，Polynoia 的富 Artifacts（可选预览/编辑的 diff 卡、web 预览、表格）是更好的交互体验。
3. **5 层 Context Assembler** —— CJK-aware token 估算 + 2-pass 预算强制比当前简单的上下文拼接更精细。
4. **双模式沙箱 + 引导式冲突解决** —— 多 Agent 并行工作时的冲突管理是真实需求。

**差距：**
- Polynoia P0 没有 CPU/RAM 隔离（计划 P1+ 中），Hermes 在安全隔离方面有类似的机会
- Polynoia 的 memory 体系（5 层 Assembler）与 Hermes 的 TencentDB 四层记忆系统各有侧重，前者更关注单次会话的 token 预算，后者关注跨会话记忆持久化

### 整合可能性

Polynoia 的 **PAP 协议**（11 种 AdapterEvent）理论上可扩展为 Hermes 的一个 Adapter，让 Hermes Agent 作为 polynoia 群聊中的一个 Agent 成员。但现阶段意义不大——两者的设计理念差异较大（Polynoia 是 IM 优先的协作平台，Hermes 是个人 Agent 运营中心），整合的成本高于收益。

更有价值的是**理念层面的借鉴**：将 Polynoia 的 Orchestrator 状态机、Context Assembler 预算策略、Conflict First 思维融入 Hermes 的未来演进。

## 归档日志

- 2026-07-20 归档
