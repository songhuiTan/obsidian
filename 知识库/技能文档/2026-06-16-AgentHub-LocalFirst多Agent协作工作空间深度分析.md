---
title: "AgentHub — Local-First 多 Agent 协作工作空间（深度源码分析）"
source: "lizyoko9/bitdance-agenthub (GitHub)"
source_url: "https://github.com/lizyoko9/bitdance-agenthub"
date: "2026-06-16"
tags: [AI, Agent, 多Agent协作, 开源, Next.js, Electron, MCP, 本地优先, 架构分析, Orchestrator]
---

# AgentHub — Local-First 多 Agent 协作工作空间

> 把多 Agent 协作做成 IM 群聊体验 — Agent 是"联系人"，对话是"工作空间"，Orchestrator 是"群里的项目经理"。

**仓库：** <https://github.com/lizyoko9/bitdance-agenthub>
**完整分析日期：** 2026-06-16（从 specs/ 17 份规格文档 + src/ ~235 个源码文件综合得出）

---

## 一、项目定位

AgentHub 解决的问题：现有 Coding Agent 要么是单次终端会话（孤立、无记忆），要么依赖托管云服务。真实协作需要**跨会话、跨 Agent、跨设备**的工作流。

AgentHub 的答案：**本地优先（Local-First）、IM 风格、多 Agent 编排的工作空间。**

| 维度 | 传统终端式 | AgentHub |
|------|-----------|----------|
| 会话模型 | 单次对话，结束后消失 | IM 群聊，永久保存 |
| Agent 管理 | 一个模型/工具集 | 多个 Agent 可切换、定制、组队 |
| 工作空间 | 当前目录 | 每个会话有独立沙箱/本地绑定 |
| 编排 | 手动指挥 | Orchestrator 自动拆任务、DAG 调度、冲突检测 |
| 产物 | 终端输出 | 6 种结构化 Artifact（web_app/document/ppt/image/code_file/diff） |
| 安全 | 无限制 | fs_write 审批、Bash 黑名单/审批、沙箱隔离 |
| 设备 | 桌面终端 | Web + Electron 桌面 + 手机端伴随 App |
| 适配器 | 单一 | Claude Code / Codex / Custom (OpenAI 兼容) / Mock |

**状态：** 活跃开发中，五层架构完整落地、功能闭环已跑通。~100+ commit 演进。Web 可用，Electron 桌面打包（DMG/EXE）完成，手机端伴随 App 脚手架已搭。

---

## 二、五层架构深度拆解

```
L5  UI 组件层          React + shadcn/ui + CodeMirror     ~50 个组件
L4  状态 + 传输层       Zustand + Immer + SSE 单连接       app-store (1269 行)
L3  应用服务层          AgentRunner / EventBus / 10+ Service
L2  Agent 适配器层      Claude Code / Codex / Custom / Mock    4 种
L1  持久化层            SQLite (better-sqlite3) + Drizzle ORM   9 张表
```

---

### L1 — 持久化层

**位置：** `src/db/`（13 个文件）

**数据库：** SQLite（better-sqlite3）+ Drizzle ORM，WAL 模式，HMR-safe 单例。

**9 张表结构：**

| 表 | 前缀 | 关键字段 | 备注 |
|----|------|---------|------|
| `agents` | `ag_` | name, adapterName, modelProvider, systemPrompt, toolNames, isOrchestrator | 内置 5 个 Agent |
| `conversations` | `conv_` | mode(single/group), agentIds, pinnedMessageIds, fsWriteApprovalMode | **聚合根** |
| `messages` | `msg_`/`msg_err_` | role, parts[] (JSON), status, parentMessageId, runId | **parts 数组** |
| `artifacts` | `art_` | type, content (JSON), version, parentArtifactId | 版本链 |
| `workspaces` | `ws_` | mode(sandbox/local), rootPath, boundPath | 1:1 与 Conversation |
| `attachments` | `att_` | kind(image/file), filePath, size | - |
| `agent_runs` | `run_` | status, parentRunId, usage (JSON) | - |
| `context_summaries` | - | compressed context | 手动压缩 |
| `app_settings` | - | 单行表 | API keys, companion mode 等 |

**数据文件：** `.agenthub-data/agenthub.db`
**Workspace 文件：** `.agenthub-data/workspaces/<convId>/`
**桌面打包后：** `~/Library/Application Support/AgentHub/data/agenthub.db`

**增量迁移：** 10 个迁移脚本（usage / bookmarks / workspace-mode / app-settings / FTS5 search / read-attachment 等），非 drizzle-kit CLI，用 raw SQL。

**设计决策：**
- Conversation 是聚合根，级联删除
- Agent 删除不级联到消息（保留历史）
- 消息内容不存纯文本，存 `parts[]` JSON 数组
- Artifact 用 JSON 列存结构化内容，不走文件系统（除 code_file）

---

### L2 — Agent 适配器层

**位置：** `src/server/adapters/`（9 个文件）

**统一接口：** `AgentPlatformAdapter`
```typescript
stream(input: AdapterInput, signal: AbortSignal): AsyncIterable<StreamEvent>
```

**4 种 Adapter 实现：**

| 适配器 | 文件行数 | SDK | 特点 |
|--------|---------|-----|------|
| **ClaudeCodeAdapter** | 792 行 | `@anthropic-ai/claude-agent-sdk` | 创建 MCP server 桥接 AgentHub 工具，canUseTool 审批桥，Session 续接 |
| **CustomAgentAdapter** | 443 行 | OpenAI SDK（通用兼容） | 自驱 tool loop（MAX_TURNS=8），处理 DeepSeek thinking 模式，多模态内容 |
| **CodexAdapter** | ~200 行 | `@openai/codex-sdk` | MCP/tool bridge，Review 模式=read-only，Auto=workspace-write |
| **MockAdapter** | ~80 行 | - | 预定义事件流，开发测试不耗 token |

**API Key 四层优先级：** agent.apiKey → app_settings → 环境变量 → OAuth（Claude Code）

**自建 Agent（UI 操作）：**
- 默认 adapterName='custom'，可选 claude-code / codex
- 自定义 systemPrompt、模型、Provider、工具集
- **不能设置为 Orchestrator**（isOrchestrator=false 写死）
- 内置 Agent 可编辑不可删除

---

### L3 — 应用服务层

这是最厚的一层，位于 `src/server/`，约 50 个文件。

#### 3a. 核心引擎：AgentRunner（2599 行！）

**位置：** `src/server/agent-runner.ts`

这是整个系统最核心的文件。主要流程：

```typescript
AgentRunner.run(args) → { runId, promise }
  └─ executeRun()
       ├─ executeSimpleRun()           // 非 Orchestrator
       │    ├─ resolveAdapter(agent)
       │    ├─ adapter.stream(input)   // → AsyncIterable<StreamEvent>
       │    └─ consumeStream(events)   // 持久化 + 广播
       │
       └─ executeOrchestratorRun()     // Orchestrator
            ├─ PHASE 1: PLAN
            │    ├─ read-only tools + plan_tasks
            │    └─ → dispatch.plan.pending
            ├─ PHASE 2: REVIEW 门控
            │    ├─ approve → 执行规划
            │    ├─ reject → 取消
            │    └─ revise → 重新 PLAN
            ├─ PHASE 3: EXECUTE
            │    ├─ Semaphore(4) 并发控制
            │    ├─ DAG 依赖解析（dependsOn）
            │    ├─ 同波次代码冲突检测（detectWaveConflicts）
            │    ├─ 动态重规划（max 4 轮）
            │    └─ 跳过传播（上游失败则下游跳过）
            └─ PHASE 4: AGGREGATE
                 └─ 收集结果 → Orchestrator 汇总回答
```

**关键常量：** `MAX_DISPATCH_ROUNDS=4`, `MAX_CHILD_TASK_ATTEMPTS=4`, `MAX_CONCURRENT_SUB_AGENT_RUNS=4`

**Orchestrator 核心设计：**
1. Orchestrator 是普通 Agent，但配了 `plan_tasks` 工具 + 特殊 system prompt
2. 执行走的同一套 AgentRunner，只是走 `executeOrchestratorRun` 分支
3. PLAN 阶段限制为只读工具（不能写盘）
4. 子 Agent 得到隔离的上下文（非完整会话历史）

#### 3b. 事件总线：EventBus

**位置：** `src/server/event-bus.ts`（40 行）

- 包装 `EventEmitter` 的进程级单例
- HMR-safe：`globalThis.__agenthubEventBus`
- `publish(event)` / `subscribe(listener) => unsubscribe()`

#### 3c. StreamEvent 协议（核心契约）

**位置：** `src/shared/types.ts`（466 行，定义 30+ 事件类型）

**这是粘合全系统的核心契约**——每一层都通过 StreamEvent 通信。

| 事件类别 | 具体事件 |
|---------|---------|
| Run 生命周期 | `run.start`, `run.end`, `run.usage` |
| Message 生命周期 | `message.start`, `message.end`, `message.added`, `message.removed` |
| Part 流式 | `part.start`, `part.delta`（text/code/thinking），`part.end` |
| Tool 调用 | `tool.call`, `tool.result` |
| Artifact | `artifact.create`, `artifact.update`, `deploy.status` |
| Orchestrator 调度 | `dispatch.plan`, `dispatch.start`, `dispatch.end`, `dispatch.plan.pending`, `dispatch.plan.resolved` |
| 审批 | `fs_write.pending/resolved`, `bash_command.pending/resolved` |
| 心跳 | `heartbeat`（15s） |

**设计原则：** 细粒度事件、增量 delta、传输无关（当前 SSE，未来可 WebSocket）。

#### 3d. 完整数据流（从用户输入到 UI 渲染）

```
用户发消息
    ↓
ChatPanel → lib/api.sendMessage() → POST /api/conversations/[id]/messages
    ↓
conversation-service.sendMessage()
  ├─ 插入用户 MessageRow → DB
  ├─ 广播 message.added → EventBus
  ├─ 确定响应 Agent（单聊/群聊 @mention）
  └─ 调用 AgentRunner.run() 对每个响应者
    ↓
AgentRunner.run()
  ├─ 创建 AbortController
  ├─ 从 DB 取 agent/workspace
  ├─ 解析 attachments
  └─ executeSimpleRun() 或 executeOrchestratorRun()
    ↓
AgentRegistry.getAdapter(agent) → Adapter.stream()
  ├─ CustomAdapter: tool loop + LLM 流
  ├─ ClaudeCodeAdapter: SDK query() 桥
  └─ → AsyncIterable<StreamEvent>
    ↓
consumeStream()
  ├─ 遍历 Adapter 产出的事件流
  ├─ persistEvent(): DB 写入（消息/产物/run/usage）
  ├─ publish() → EventBus → EventEmitter
  ├─ 注入 artifact_ref parts（对 artifact.create 事件）
  └─ 处理 onToolCall 拦截（如 plan_tasks 暂停）
    ↓
EventBus → SSE /api/stream 路由
  └─ readableStream → "data: {json}\n\n" → HTTP Response
    ↓
StreamProvider（客户端 EventSource）
  └─ onmessage → JSON.parse → store.applyEvent(event)
    ↓
Zustand app-store.applyEvent()（1269 行）
  ├─ switch(event.type) → immer 更新状态
  ├─ message.start → 创建流式消息
  ├─ part.delta → 追加文本/思考内容
  ├─ tool.call → 推 tool_use part
  ├─ tool.result → 推 tool_result part
  ├─ artifact.create → 加入 artifacts
  └─ fs_write.pending → 加入审批队列
    ↓
React 组件从 Zustand selector 重渲染
  ├─ MessageList 显示流式文本
  ├─ ArtifactPreviewPanel 显示产物卡
  └─ PendingWritesPanel 显示审批弹窗
```

#### 3e. 10+ 应用服务

| 服务 | 文件 | 行数 | 职责 |
|------|------|------|------|
| ConversationService | `conversation-service.ts` | 1021 | 会话 CRUD + sendMessage（核心入口） |
| AgentService | `agent-service.ts` | ~120 | Agent CRUD |
| SettingsService | `settings-service.ts` | ~100 | 三层 Key 优先级解析 |
| SearchService | `search-service.ts` | ~150 | FTS5 全文搜索 |
| ArtifactService | `artifact-service.ts` | ~200 | Artifact CRUD + 版本管理 |
| DeploymentService | `deployment-service.ts` | ~150 | 部署生命周期 |
| FsService | `fs-service.ts` | ~100 | 文件读写 |
| ContextCompactionService | `context-compaction-service.ts` | ~200 | 会话压缩为摘要 |
| SecurityService | `security.ts` | ~100 | Bash 黑名单模式 |
| DispatchPlan | `dispatch-plan.ts` | ~300 | 规划验证 + 依赖解析 + 重规划上下文 |
| DispatchFileWrites | `dispatch-file-writes.ts` | ~200 | 文件冲突检测 |
| BashCommandApproval | `bash-command-approval.ts` | ~80 | Bash 安全等级分类 |

**审批 gate（4 种 in-memory pending queue）：**
- `pending-writes.ts` — fs_write 审批
- `pending-bash-commands.ts` — Bash 命令审批
- `pending-questions.ts` — ask_user 结构化问题
- `pending-dispatch-plans.ts` — Orchestrator 计划审查

---

### L4 — 状态 + 传输层

**位置：** `src/stores/app-store.ts`（1269 行）

**Zustand + Immer 状态结构：**

```
entities:
  conversations: Record<string, ConversationWithMeta>
  agents:        Record<string, AgentRow>
  messages:      Record<string, MessageRow>
  artifacts:     Record<string, ArtifactRow>

relations:
  messageIdsByConv:   Record<string, string[]>
  runsByConv:         Record<string, Record<string, AgentRunRow>>
  dispatchesByRunId:  Record<string, DispatchState>

pending:
  pendingWritesByConv       / pendingBashCommandsByConv
  pendingQuestionsByConv    / pendingDispatchPlansByConv

UI state:
  activeConversationId, previewArtifactId, fileExplorerOpen
  openFilesByConv, activeTabByConv, replyTargetByConv
  streamConnected, etc.
```

**派生 hooks（通过 Zustand selector）：**
- `useMessagesForConversation(id)`
- `usePinnedMessagesForConversation(id)`
- `useActiveConversation()`
- `useConversationList()`
- `useTopLevelRunningRuns()`
- `usePendingPlanReviewForConversation(id)`

**SSE 传输：** `src/components/stream-provider.tsx`（70 行）
- 全局单连接到 `/api/stream`
- 15s 心跳保活
- React 19 StrictMode 双挂载兼容（refCount）
- `onmessage → JSON.parse → store.applyEvent(event)`

---

### L5 — UI 组件层

**位置：** `src/components/`，约 50 个组件

**主布局：** `page.tsx` → `<Sidebar /> <ChatPanel /> <FileExplorerPanel /> <ArtifactPreviewPanel />`

**侧栏：**
| 组件 | 行数 | 职责 |
|------|------|------|
| `sidebar.tsx` | ~200 | 4 Tab（会话/产物库/Agents/分析） |
| `agent-library.tsx` | ~150 | Agent 列表管理 |
| `artifact-library.tsx` | ~150 | 产物浏览 |
| `global-search.tsx` | ~100 | ⌘K 全局搜索 |

**聊天面板：**
| 组件 | 行数 | 职责 |
|------|------|------|
| `chat-panel.tsx` | 351 | 主面板（Tab 系统 + 浮层） |
| `message-list.tsx` | ~150 | 消息列表（自动滚动 + 回复高亮） |
| `message-item.tsx` | ~200 | 单条消息（角色/Agent 头像/操作按钮） |
| `message-parts.tsx` | ~200 | 渲染 10 种 MessagePart |
| `message-input.tsx` | ~250 | 富输入（@mention / 附件/ 斜杠命令） |
| `markdown.tsx` | ~100 | 自定义 Markdown 渲染 |
| `code-block.tsx` | ~80 | 语法高亮代码块 |

**特色面板：**
| 组件 | 行数 | 职责 |
|------|------|------|
| `artifact-preview-panel.tsx` | ~200 | 右面板产物预览 |
| `dispatch-plan-card.tsx` | ~200 | Orchestrator 调度卡 |
| `pending-writes-panel.tsx` | ~200 | fs_write 审批 + diff |
| `pending-bash-commands-panel.tsx` | ~100 | Bash 审批 |
| `ask-user-question-dialog.tsx` | ~80 | 结构化问题弹窗 |
| `file-explorer-panel.tsx` | ~200 | 文件浏览/编辑 |
| `file-tab.tsx` | ~150 | 文件编辑（CodeMirror） |
| `settings-dialog.tsx` | ~200 | API key/端点设置 |
| `usage-dashboard.tsx` | - | Token 计量分析 |
| `conversation-outline.tsx` | ~100 | 书签导航 |

---

## 三、Message Parts 系统（10 种）

**消息不再是纯文本，而是 parts 数组**——这是 AgentHub 产品体验的核心设计。

| Part | 增量 | 渲染 | 说明 |
|------|------|------|------|
| `text` | ✅ text.append | `<Markdown>` | 流式文本 |
| `code` | ✅ code.append | `<CodeBlock>` | 流式代码 |
| `thinking` | ✅ thinking.append | `<ThinkingPart>`（可折叠） | LLM 推理过程 |
| `tool_use` | ❌ | `<ToolUsePart>`（与 tool_result 合并） | 工具调用声明 |
| `tool_result` | ❌ | 被 ToolUsePart 吸收 | 工具执行结果 |
| `artifact_ref` | ❌ | `<ArtifactRefPart>`（懒加载） | Artifact 引用 |
| `deploy_status` | ❌ | `<DeployStatusPart>` | 部署状态 |
| `deploy_candidates` | ❌ | `<DeployCandidatesPart>` | 部署候选 |
| `image_attachment` | ❌ | `<AttachmentChip>` | 图片附件 |
| `file_attachment` | ❌ | `<AttachmentChip>` | 文件附件 |

**历史序列化：** parts → LLM-readable text（tool_use/tool_result/thinking 被丢弃，artifact_ref 折叠为 `[产物: title (id=...)]` 占位）

---

## 四、工具系统（12 个内置工具）

**位置：** `src/server/tools/`

**注册方式：** `ToolRegistry` 维护名称→`ToolDef` 映射。`ToolDef` 包含 JSON Schema parameters。

| 工具 | 副作用 | 目标 Agent | 说明 |
|------|--------|-----------|------|
| `write_artifact` | DB write | 交付 Agent | 创建 6 种 Artifact，规范 4 种内容格式 |
| `read_artifact` | DB read | 所有 Agent | 按 ID 读取 Artifact |
| `read_attachment` | FS read | 文档 Agent | 读取附件 |
| `deploy_artifact` | FS + publish | Web App Agent | 从 Artifact 生成预览部署 |
| `deploy_workspace` | FS + publish | 代码 Agent | 从 workspace 目录生成部署 |
| `plan_tasks` | 无（输出工具） | **仅 Orchestrator** | 输出任务规划 DAG |
| `report_task_result` | 无（输出工具） | **仅子 Agent** | 报告子任务结果 |
| `fs_list` | FS read | 代码 Agent | 列出 workspace 文件 |
| `fs_read` | FS read | 代码 Agent | 读取文件（path 校验） |
| `fs_write` | FS write | 代码 Agent | 写文件（审批门控） |
| `bash` | Process/FS | 代码 Agent | 执行命令（安全+审批） |
| `ask_user` | 内存（pending） | 所有 Agent | 结构化多选问题 |

**安全控制：**
- Bash：30s 超时 / 10K 字符截断 / 双平台黑名单 / 风险等级分类
- fs_write：Review/Auto 模式 / per-conversation 设定
- path：所有文件操作限定在 effective workspace
- Artifact iframe：沙箱需 `allow-scripts` 无 `allow-same-origin`

---

## 五、API 路由系统（52 个路由处理器）

**位置：** `src/app/api/`

| 路由组 | 方法 | 作用 |
|-------|------|------|
| `/api/stream` | GET | **SSE 全局事件流**（单连接） |
| `/api/conversations` | GET/POST | 列表/创建 |
| `/api/conversations/[id]` | PATCH/DELETE | 重命名/添加Agent/归档/审批模式 |
| `/api/conversations/[id]/messages` | GET/POST/DELETE | 消息 CRUD |
| `/api/conversations/[id]/pending-*` | GET/POST | 4 种审批 gate |
| `/api/conversations/[id]/fs/*` | POST | 文件操作 |
| `/api/conversations/[id]/compact` | POST | 上下文压缩 |
| `/api/conversations/[id]/regenerate` | POST | 重新生成 |
| `/api/agents` | GET/POST | Agent 列表/创建 |
| `/api/agents/[id]` | PATCH/DELETE | 修改/删除 |
| `/api/agents/draft` | POST | AI 生成 Agent 草稿 |
| `/api/artifacts` | GET/POST | 产物 CRUD |
| `/api/artifacts/[id]/versions` | GET/POST | 版本管理 |
| `/api/messages/[id]/{edit,withdraw,pin,bookmark}` | POST | 消息操作 |
| `/api/search` | GET | FTS5 全文搜索 |
| `/api/settings` | GET/PATCH | 全局设置 |
| `/api/runs/[id]/abort` | POST | 中止 run |
| `/api/usage/summary` | GET | Token 统计 |
| `/api/mobile/*` | - | 手机端 API（8 个端点） |

---

## 六、安全模型

AgentHub 假设 LLM 输出不可信。

- **文件沙箱**：所有 fs/bash 工具解析路径到 effective workspace
- **Bash 黑名单**：双平台（POSIX/Windows）独立黑名单
- **审批门控**：4 种审批类型（文件写 / Bash 命令 / 用户问题 / 调度计划）
- **Artifact iframe**：沙箱 `allow-scripts` 无 `allow-same-origin`
- **路径安全**：拒绝 UNC 路径 / 符号链接循环检测
- **子进程清理**：Windows 用 `taskkill /F /T /PID` / POSIX 发送 SIGTERM 后 SIGKILL
- **API Keys**：纯本地配置，无托管 Key 服务

**已知限制：**
- Claude Code SDK 写盘绕过 AgentHub 沙箱配额
- 部分 SDK 的命令/文件审批桥依赖底层适配器暴露程度

---

## 七、平台抽象（跨平台）

**位置：** `src/server/platform.ts` + security.ts

| 维度 | POSIX | Windows |
|------|-------|---------|
| Shell | 用户 login shell（zsh/bash `-l -i -c`） | PowerShell 5.1 |
| 黑名单 | rm -rf /, dd, > /dev/sda 等 | 相同的逻辑模式 |
| 进程清理 | kill -TERM →SIGKILL | taskkill /F /T /PID |
| 路径 | 无问题 | 多盘符 DirPicker |
| Workspace 清理 | rm -rf | 指数退避重试 |

---

## 八、Orchestrator 编排（核心差异化能力）

### 完整流程

```
用户: "帮我重构用户模块"
    ↓
Orchestrator LLM 被调用（带 plan_tasks 工具）
    ↓
PHASE 1 — PLAN
  Orchestrator 调用 plan_tasks → 输出 DAG 规划
  DAG 结构: [{taskId, description, agentId, dependsOn, expectedOutputs, inputs}]
    ↓
PHASE 2 — REVIEW（门控）
  用户审查规划卡片
  ├─ Approve → 锁定规划，进入 PHASE 3
  ├─ Reject → 取消整个调度
  └─ Revise → 自然语言反馈 → Orchestrator 重新规划（max 2 轮）
    ↓
PHASE 3 — EXECUTE
  Semaphore(4) 控制并发
  Wave 1: 无依赖的 task 并行执行
  Wave 2: 依赖 Wave 1 结果的 task
  ...
  同波次冲突检测: hash-based, detectWaveConflicts()
  失败传播: 上游失败 → 下游跳过
  动态重规划: 失败/冲突 → 触发 max 4 轮重规划
    ↓
PHASE 4 — AGGREGATE
  收集所有子 Agent 的 report_task_result
  Orchestrator LLM 汇总成最终回答
```

### 子 Agent 上下文隔离

子 Agent 不接收完整会话历史。它们得到的是：
- **`<required_inputs>`** XML 块（上游 task 的 outputKey）
- **`<expected_outputs>`** XML 块（本 task 的预期产出）
- **`<acceptance_criteria>`** XML 块（验收标准）
- `report_task_result` 必须调用，可选传 `acceptanceResults`

### 冲突检测

- 追踪每个子 run 的 fs_write 写入集合（文件路径 + hash）
- 同波次任务写入相同文件 → `detectWaveConflicts()` 上报
- **不自动合并**，仅检测 + 上报（Spec 06 明确 "No auto-merge"）
- 盲区：bash / SDK adapter 写入不被追踪

---

## 九、Artifact 产物系统

**6 种类型，同一 DB 表（JSON 列）：**

| 类型 | 存储 | 预览 | 版本链 |
|------|------|------|--------|
| `web_app` | DB JSON（files: Record<path, source>） | iframe 沙箱 + 预览 URL | parentArtifactId |
| `code_file` | Workspace 路径引用 | CodeMirror 编辑 | parentArtifactId |
| `document` | DB JSON（markdown） | Markdown 渲染 | parentArtifactId |
| `image` | DB JSON（url + alt） | 图片预览 | parentArtifactId |
| `diff` | DB JSON（legacy） | 只读兼容 | 只读 |
| `ppt` | DB JSON（slides + blocks） | 分页预览 + 真 .pptx 导出 | parentArtifactId |

**设计要点：**
- `write_artifact` 工具将 4 种内容格式规范化为统一结构（容忍 LLM 格式漂移）
- `deploy_artifact` 可为 web_app 生成本地预览 URL + 源码包/容器包
- AgentHub 注入 `artifact_ref` parts（Adapter 不需要知道产物的存在）
- `diff` 类型不再由 Agent 创建（被版本对比替代）

---

## 十、Electron 桌面 + 手机端

### 桌面（Electron 33）

**位置：** `electron/`（main.ts + paths.ts + server-bootstrap.ts）

- **架构：** Electron 主进程内部启动 Next.js standalone（非 spawn），渲染器通过 HTTP/SSE 连接 `127.0.0.1:<random-port>`
- **数据库路径：** `app.getPath('userData')/data/agenthub.db`
- **ABI 策略：** 两套 better-sqlite3 编译（Node ABI 开发/测试，Electron ABI 130 打包）
- **打包：** macOS DMG（arm64）/ Windows NSIS（x64）
- **注意：** `asarUnpack: [".next/standalone/**"]` 必须，`npmRebuild: false`

### 手机端（Capacitor）

**位置：** `apps/mobile/`

- **设计：** 伴随客户端（非独立运行时）
- **模式：** companion mode（off / lan / tailnet），设备 token 配对
- **通信：** bearer token auth + fetch streaming
- **手机不做：** Agent 运行 / LLM 调用 / DB 写入 / 文件操作
- **状态：** 响应式 Web 已适配，Capacitor 原生壳脚手架已搭，配对通信待打通

---

## 十一、测试体系

| 层次 | 工具 | 覆盖内容 |
|------|------|---------|
| 单元测试 | Vitest | security, workspace-utils, dispatch-plan, artifact-content, ppt-export, ppt-theme |
| E2E | Playwright | 核心 IM 流（mock agent），基建已搭 |

**待补：** 产物预览/导出 + 群聊调度 E2E（需测试假 adapter）

---

## 十二、文档体系

| 文档 | 用途 | 读者 |
|------|------|------|
| `README.md` | 安装/快速开始 | 人类用户 |
| `OVERVIEW.md` | 全貌速览 + 代码地图 + 进度 | AI / 新开发者 |
| `CLAUDE.md` | 协作规则/约束 | AI 协作者 |
| `specs/`（17 份） | 编号规格 + 字段契约 | 开发者（改动必读） |
| `openspec/` | OpenSpec 能力索引 | 开发者 |
| `skills/` | 扩展任务步骤化指南 | AI 协作者 |
| `docs/AI-COLLABORATION.md` | 协作方法论 + 实录 | 开发者 |

**核心契约（改动必同步）：**
- 改实体字段 → `specs/01`
- 改事件 → `specs/02`
- 改 Bash 黑名单 → `specs/11` + `src/server/security.ts`
- 所有 LLM 调用必带 `AbortSignal`
- 跨进程输入必过 zod 校验
- fs/bash 必过 Workspace 沙箱

---

## 十三、与同类项目对比

| 维度 | AgentHub | Cline | Claude Code / Codex |
|------|----------|-------|---------------------|
| 形态 | 完整桌面 App | VS Code 扩展 | CLI |
| 多 Agent | ✅ 原生 + Orchestrator | ❌ 单 Agent | ❌ 单 Agent |
| 会话模型 | IM 群聊持久化 | 单次对话 | 单次终端 |
| 设备覆盖 | Web + 桌面 + 手机 | 桌面扩展 | CLI |
| 安全 | 沙箱 + 4 种审批 + 黑名单 | 扩展依赖宿主 | SDK 自有 |
| 产物 | 6 种结构化 Artifact | 文件输出 | 文件输出 |
| 代码量 | ~235 源文件 | - | - |
| 协议 | 未声明（学习研究项目） | Apache 2.0 | 闭源 |

AgentHub 最独特的价值：**将多 Agent 协作产品化**——不仅仅是多模型并联，而是完整的会话模型、任务编排、安全边界和设备生态。

---

## 十四、源码文件统计

| 目录 | 文件数 | 关键行数 | 说明 |
|------|--------|---------|------|
| `src/shared/` | 10 | ~900 | 类型定义 + 常量 + 工具函数 |
| `src/db/` | 13 | ~800 | Schema + 迁移 + 种子数据 |
| `src/server/` | ~30 | ~6500 | **核心引擎** |
| `src/server/adapters/` | 9 | ~1900 | 4 种 Adapter 实现 |
| `src/server/tools/` | 13 | ~1500 | 12 个内置工具 |
| `src/app/api/` | ~52 routes | ~3000 | API 路由处理器 |
| `src/stores/` | 4 | ~1400 | Zustand 状态管理 |
| `src/components/` | ~50 | ~5000 | UI 组件 |
| `src/lib/` | 8 | ~1200 | 客户端 API + 工具 |
| `electron/` | 3 | ~300 | 桌面端 |
| `specs/` | 17 | ~10000 | 规格文档（设计契约） |

**总计：** ~235 源文件 + 17 份规格文档

---

## 十五、架构价值评估

### 最强优势

1. **事件驱动架构干净** — StreamEvent 统一协议贯穿全系统，每一层通过事件通信
2. **Orchestrator 编排产品化程度高** — 规划→门控→DAG→冲突检测→汇总，闭环完整
3. **Adapter 抽象层优雅** — Claude Code / Codex / Custom 统一接口，可插拔
4. **本地优先符合趋势** — SQLite + 本地沙箱，无需托管，隐私友好
5. **安全模型完整** — 4 种审批门控 + 黑名单 + 沙箱，单用户场景已足够

### 可借鉴的设计

1. **Parts 化消息结构** — 不是纯文本而是结构化数组，极大丰富渲染和上下文控制
2. **五级文档体系** — 规则/地图/规格/配方/方法论分离
3. **冲突检测在编排层** — 而非文件系统层，更加语义化
4. **审批门控 Promise 模式** — in-memory pending queue 简洁高效
5. **HMR-safe 单例** — globalThis 模式解决开发热重载问题

### 注意点

1. 缺正式开源协议
2. SQLite + Electron ABI 切换增加运维复杂度
3. Claude Code SDK 写盘绕过沙箱（架构级问题）
4. 多 Agent 并发 token 成本未文档化
5. 规划审查的「修正」走自然语言反馈，可能精度不够

---

## 十六、关键学习点（对你项目的启发）

1. **StreamEvent 协议设计**：用一个联合类型贯穿全系统，前端后端共享同一份类型定义，避免 DTO 层
2. **Adapter 模式**：将不同 Agent SDK 统一到 `AsyncIterable<StreamEvent>` 接口，可扩展性好
3. **审批门控模式**：Promise-based pending queue，简洁、可等待、可超时
4. **Parts 化消息**：消息内容不再是字符串而是结构化数组，前端按 type 分派渲染组件
5. **文档即契约**：specs/ 设计文档与代码同步更新，改代码前先读 spec
