---
title: Multica：一周涨 12K Star，Go 如何把 AI Agent 变成你的「项目队友」
author: Go语言中文网
date: "2026-04-20"
source: "https://mp.weixin.qq.com/s/ZZyA_iLk8awhyWb2OEBrVA"
---

# Multica：一周涨 12K Star，Go 如何把 AI Agent 变成你的「项目队友」

当 Claude Code、Codex、OpenCode 还在被当"工具"用的时候，Multica 已经把它们放上了项目看板——有头像、有名字、能领任务、会报告进度。这个 Go + Next.js 的 AI 原生项目管理平台，Go 后端 32,000 行代码实现了 WebSocket Hub 实时推送、事件总线解耦、sqlc 类型安全查询、Daemon 轮询执行引擎。Agent Backend 接口统一了 5 种 Agent 的通信协议，双态作者系统让人类和 Agent 在数据模型中完全对等。Devv.AI 联合创始人作品，一周涨 12k Star。

## 从"工具"到"队友"：AI Agent 的进化

当前 AI 编程 Agent 的使用方式存在一个根本问题：**Agent 是工具，不是团队成员**。

你打开终端，输入 `claude` 或 `codex`，Agent 帮你写代码，关掉终端就结束了。没有持续的任务上下文，没有项目看板，没有进度追踪，没有技能沉淀。更关键的是——当团队中有多个 Agent 同时工作时，谁来协调？谁先做什么？谁卡住了？

Multica 的回答是：**把 Agent 放到项目管理里，让它成为真正的队友**。

名字 Multica 致敬 1960 年代的 Multics 操作系统——"多路复用信息和计算代理"。当年 Multics 让多个用户共享一台大型机，今天 Multica 让多个 Agent 共享一个项目。目标是：2 名工程师 + 一组 Agent = 20 人团队的推进速度。

## 项目概览

Multica 是一个 AI 原生的项目管理平台，核心创新是让 AI Agent（Claude Code、Codex、OpenCode 等）成为项目看板上的"队友"。

| 指标 | 数值 |
| --- | --- |
| GitHub Stars | 16.4k+ |
| 开源协议 | AGPL-3.0 |
| 后端语言 | Go 1.26.1 |
| 前端 | Next.js 16 (App Router) |
| 数据库 | PostgreSQL 17 + pgvector |
| Go 后端代码量 | 32,354 行（151 个 .go 文件） |
| 数据库迁移 | 39 个 |
| 支持 Agent | 5 种（Claude Code、Codex、OpenCode、OpenClaw、Hermes） |
| sqlc 查询文件 | 22 个，约 1100 行 SQL |
| 创始人 | Bohan Jiang（Devv.AI 联合创始人） |

## 系统架构：四层协作

```
┌──────────────┐     ┌──────────────┐     ┌──────────────────┐
│   Next.js    │────>│  Go Backend  │────>│   PostgreSQL     │
│   Frontend   │<────│  (Chi + WS)  │<────│   (pgvector)     │
└──────────────┘     └──────┬───────┘     └──────────────────┘
                            │
                     ┌──────┴───────┐
                     │ Agent Daemon │  ← 运行在你的机器上
                     │Claude/Codex/ │
                     │OpenClaw/Code │
                     └──────────────┘
```

四层各司其职：

1. **前端（Next.js）**：任务看板、Agent 管理、实时流式输出、Chat 对话
2. **Go 后端（Chi + WebSocket）**：REST API + 实时推送 + 事件总线 + 权限控制
3. **PostgreSQL**：多工作空间隔离、双态作者系统、pgvector 预留向量搜索
4. **Agent Daemon**：本地运行，轮询后端领取任务，启动 Agent CLI 执行

这个架构的关键决策是 **Daemon 运行在本地而非云端**——你的代码永远不会离开你的机器，Agent 在本地环境中执行，结果通过 API 上报到 Multica 后端。

## Go 后端：工程模式解析

### Chi 路由与中间件栈

路由定义在 `server/cmd/server/router.go`，采用 Chi 的分组模式：

```
// 中间件栈
r.Use(middleware.RequestID)
r.Use(middleware.RequestLogger)    // 请求日志
r.Use(middleware.Recoverer)        // panic 恢复
r.Use(cors.Handler(corsOptions))  // CORS

// 公开路由
r.Post("/auth/send-code", h.SendCode)
r.Post("/auth/verify-code", h.VerifyCode)

// Daemon 路由（Daemon Token 或 User Token 认证）
r.Route("/api/daemon", func(r chi.Router) {
    r.Post("/register", h.Register)
    r.Post("/heartbeat", h.Heartbeat)
    r.Post("/task/{id}/claim", h.ClaimTask)
    r.Post("/task/{id}/status", h.UpdateStatus)
})

// 受保护路由（JWT/PAT 认证）
r.Route("/api", func(r chi.Router) {
    r.Use(h.AuthMiddleware)
    r.Use(h.RequireWorkspaceMember)  // 工作空间成员检查
    // Issue、Agent、Skill、Runtime、Chat 等端点
})
```

权限模型采用 **owner/admin/member 三级角色**，通过 `RequireWorkspaceRole` 中间件实现细粒度控制。支持两种认证方式：JWT（Web 登录）和 Personal Access Token（`mul_` 前缀，用于 API 调用和 WebSocket 连接）。

### WebSocket Hub：房间模型

WebSocket 实现在 `server/internal/realtime/hub.go`，采用经典的 **房间模型**：

```
type Hub struct {
    mu       sync.RWMutex
    rooms    map[string]map[*Client]bool// workspaceID -> clients
    register chan *Client
    leave    chan *Client
}

func (h *Hub) Run() {
    for {
        select {
        case client := <-h.register:
            // 加入房间
        case client := <-h.leave:
            // 离开房间，清理资源
        }
    }
}
```

三种消息分发方式：

| 方法 | 场景 | 实现 |
| --- | --- | --- |
| `BroadcastToWorkspace` | Issue 创建、任务状态变更 | 遍历房间内所有 Client |
| `SendToUser` | 个人通知、收件箱更新 | 按用户 ID 查找 Client |
| `Broadcast` | 系统公告 | 遍历所有房间 |

**慢客户端处理**：每个 Client 有一个带缓冲的 `send` channel（容量 256）。当 channel 满时，Hub 自动踢掉该 Client，防止一个慢客户端阻塞整个房间。这是 Go WebSocket 服务端的经典防护模式。

事件类型约 30 种（`issue:created`、`task:dispatch`、`agent:status`、`chat:message` 等），定义在 `server/pkg/protocol/events.go`。

### 事件总线：进程内发布/订阅

`server/internal/events/bus.go` 实现了一个进程内的同步事件总线：

```
type Bus struct {
    mu         sync.RWMutex
    handlers   map[string][]Handler  // eventType -> handlers
    allHandler []Handler             // 监听所有事件
}

func (b *Bus) Publish(event Event) {
    // 同步调用，按注册顺序执行
    for _, handler := range b.handlers[event.Type] {
        handler(event)  // 带 panic recovery
    }
}
```

注册了四组 Listeners：

| Listener | 职责 |
| --- | --- |
| `registerListeners` | 转发事件到 WebSocket Hub |
| `registerActivityListeners` | 写入 `activity_log` 表 |
| `registerNotificationListeners` | 创建 `inbox_item` 并推送通知 |
| `registerSubscriberListeners` | 更新 Issue 订阅者列表 |

**设计选择**：同步调用而非异步。好处是事件处理顺序确定、调试简单、不需要额外的 goroutine 管理。带 panic recovery 确保单个 handler 异常不影响其他 handler。对于单进程部署，同步模式已经足够。

### sqlc：类型安全的 SQL

Multica 使用 sqlc 从 SQL 文件生成 Go 代码，配置在 `server/sqlc.yaml`：

- 引擎：PostgreSQL，驱动 `pgx/v5`
- 查询文件：22 个 `.sql` 文件，约 1100 行
- 生成代码按领域划分：`agent.sql.go`、`issue.sql.go`、`task.sql.go` 等

sqlc 的价值在于：**SQL 就是 SQL，Go 就是 Go**。你写原生 SQL 查询，sqlc 生成类型安全的 Go 函数。没有 ORM 的抽象泄漏，没有字符串拼接的安全隐患。39 个数据库迁移文件也反映了项目的持续演进。

## Agent Backend：统一接口的设计

这是 Multica 最精妙的设计——用 Go 接口统一 5 种 Agent 的通信协议。

### Backend 接口

```
type Backend interface {
    Execute(ctx context.Context, prompt string, opts ExecOptions) (*Session, error)
}
```

每种 Agent 实现这套接口，但通信协议完全不同：

### Claude Code：JSON 流式通信

Claude Code 通过 `--output-format stream-json --input-format stream-json` 以子进程方式运行：

```
// 启动 Claude Code
cmd := exec.Command("claude",
    "--output-format", "stream-json",
    "--input-format", "stream-json",
    "--permission-mode", "bypassPermissions",
)

// 解析 stdout 的 JSON 流
// 消息类型：assistant, user, system, result
// 支持 --resume 恢复会话
```

`bypassPermissions` 意味着全自动批准所有操作——在项目管理的上下文中，这是合理的，因为 Agent 的行为已经被任务描述约束。

### Codex：JSON-RPC 2.0

Codex 的通信方式完全不同，使用 JSON-RPC：

```
// 启动 Codex
cmd := exec.Command("codex", "app-server", "--listen", "stdio://")

// 完整生命周期：
// 1. initialize
// 2. thread/start
// 3. turn/start
// 4. 等待完成，自动批准 exec/patch 请求
```

### 统一的消息类型

无论底层 Agent 使用什么协议，Daemon 都将输出标准化为统一的消息类型：

| 类型 | 含义 |
| --- | --- |
| `text` | Agent 文本输出 |
| `thinking` | Agent 思考过程 |
| `tool_use` | Agent 调用工具 |
| `tool_result` | 工具执行结果 |
| `error` | 错误信息 |
| `log` | 日志 |

以 500ms 为周期批量上报消息到服务器，服务器通过事件总线 + WebSocket 推送到前端。这种 **批量上报** 的设计平衡了实时性和网络开销。

## Daemon：轮询执行引擎

Daemon 是 Multica 的"手脚"——运行在开发者本地，负责实际执行 Agent。

### 核心参数

| 参数 | 默认值 | 说明 |
| --- | --- | --- |
| `PollInterval` | 3 秒 | 任务轮询间隔 |
| `HeartbeatInterval` | 15 秒 | 心跳上报间隔 |
| `MaxConcurrentTasks` | 可配置 | 并发任务上限 |

### 并发控制

使用信号量模式控制并发：

```
sem := make(chan struct{}, maxConcurrent)

func (d *Daemon) processTask(task Task) {
    sem <- struct{}{}         // 获取信号量
    defer func() { <-sem }()  // 释放信号量

    // 执行 Agent 任务...
}
```

这是一个经典的 Go 并发控制模式——用 buffered channel 实现信号量，比 `sync.WaitGroup` 更适合"持续消费"的场景。

### 任务取消

任务取消通过轮询实现：Daemon 每 5 秒检查一次任务状态，如果状态变为 `cancelled`，则取消 Agent 进程。虽然不是即时的，但对于项目管理场景已经足够。

### 任务生命周期

```
queued → dispatched → running → completed/failed/cancelled
```

1. **Issue 分配给 Agent** → 自动入队（`queued`）
2. **Daemon 轮询到任务** → 领取并创建隔离执行环境（`dispatched`）
3. **启动 Agent CLI** → 开始执行（`running`）
4. **Agent 完成/出错** → 上报结果（`completed`/`failed`）

评论中 `@Agent` 也能触发任务——这让 Agent 真正融入了团队的协作流程。

## 双态作者系统：人类和 Agent 完全对等

这是 Multica 数据模型最有趣的设计。

### 数据模型

```
-- 评论表
CREATETABLEcomment (
    id          UUID PRIMARY KEY,
    issue_id    UUIDNOTNULL,
    author_type VARCHARNOTNULL,  -- 'member' 或 'agent'
    author_id   UUIDNOTNULL,     -- 指向 user 或 agent 表
    body        TEXTNOTNULL,
    parent_id   UUID               -- 支持嵌套回复
);

-- 通知表
CREATETABLE inbox_item (
    id             UUID PRIMARY KEY,
    recipient_type VARCHARNOTNULL,  -- 'member' 或 'agent'
    recipient_id   UUIDNOTNULL,
    ...
);
```

`author_type` + `author_id` 的组合让人类和 Agent 在数据模型中完全对等——同一个 Issue 的时间线上，人类的评论和 Agent 的进度报告使用同一套数据结构，前端不需要区分。

### Issue 标识符

支持 `PREFIX-NUMBER` 格式（类似 JIRA 的 `PROJ-42`），这让 Agent 工作在项目上下文中更有"归属感"——不仅是完成一个任务，而是推进 `MUL-7` 这个 Issue。

## Skills：可复用的 Agent 技能

Skills 系统让 Agent 的能力可以沉淀和复用：

```
skill（技能定义）
  ├── skill_file（支撑文件：prompt 模板、配置等）
  └── agent_skill（多对多关联：哪个 Agent 掌握哪些技能）
```

技能注入方式因 Agent 而异：

| Agent | 注入方式 |
| --- | --- |
| Claude Code | 写入 `CLAUDE.md` / `AGENTS.md` |
| Codex | 写入 `CODEX_HOME/skills/` 目录 |
| 其他 | 通过 `execenv` 包注入执行环境 |

这个设计解决了"每个 Agent 都要重新教一遍"的问题——一个调试技能写好后，可以分配给所有 Agent。

## 数据库设计：39 次迁移的演进

从 39 个迁移文件可以窥见产品的演进路径：

| 迁移范围 | 核心表 | 设计亮点 |
| --- | --- | --- |
| 基础设施 | `user`, `workspace`, `member` | 多工作空间隔离，三级角色 |
| Agent 系统 | `agent`, `daemon_connection`, `daemon_token` | Agent 状态机（idle/working/blocked/error/offline） |
| 任务系统 | `issue`, `comment`, `agent_task_queue` | 双态作者、嵌套评论、任务队列 |
| 技能系统 | `skill`, `skill_file`, `agent_skill` | 可复用技能 |
| 社交功能 | `reaction`, `subscriber`, `pinned_item` | Issue 级别订阅和互动 |
| 资源追踪 | `runtime_usage`, `task_usage` | Agent 资源消耗量化 |
| Chat 系统 | `chat_session`, `chat_message` | Agent 直接对话（不经过 Issue） |
| 安全认证 | `personal_access_token`, `verification_code` | PAT + 邮箱验证码 |

广泛使用 JSONB 存储配置和元数据——这是 PostgreSQL 的经典"灵活列"模式，适合频繁变化的配置需求。

## 竞品对比

| 维度 | Multica | Claude Squad | OpenHands |
| --- | --- | --- | --- |
| 定位 | AI 原生项目管理平台 | 终端多 Agent 管理器 | 单 Agent 代码沙箱 |
| 技术栈 | Go + Next.js | Go (TUI) | Python |
| Agent 支持 | 5 种 | 仅 Claude Code | 仅 OpenHands |
| 任务管理 | 完整看板（7 状态） | 无 | 无 |
| 实时协作 | WebSocket + 事件总线 | tmux 分屏 | Web UI |
| 技能复用 | Skills 系统 | 无 | 无 |
| 部署方式 | 自托管 Web 服务 | 本地终端 | Docker |
| 多租户 | 工作空间隔离 | 无 | 无 |
| 数据隐私 | 代码留在本地 | 代码留在本地 | 沙箱执行 |

Multica 的三个独特价值：

1. **Agent 即队友**：不是"用 Agent 写代码"，而是"Agent 参与项目管理"——它有头像、名字、状态，能被 @、能领任务、能写评论
2. **Vendor-Neutral**：不绑定特定 Agent，5 种后端可插拔，新增 Agent 只需实现 `Backend` 接口
3. **自托管 + 代码本地**：数据在你自己的 PostgreSQL，代码在你自己的机器上

## Go 工程总结

从 Go 工程角度看，Multica 是"现代 Go Web 服务"的实践范例：

**Chi + sqlc + pgx 三件套**：放弃重量级 ORM，用 Chi 做路由、sqlc 做 SQL 代码生成、pgx 做驱动。这是 2026 年 Go 社区越来越主流的选择——性能好、类型安全、SQL 可控。

**WebSocket Hub 房间模型**：`map[workspaceID]map[*Client]bool` 的二级 map 实现多工作空间隔离，慢客户端自动踢出防止背压。经典的 Go 并发模式——channel + select + goroutine。

**事件总线解耦**：Handler 层不直接调 WebSocket、不直接写日志——发布事件，由注册的 Listeners 各自处理。这让横切关注点（通知、日志、实时推送）完全解耦。

**信号量并发控制**：`make(chan struct{}, maxConcurrent)` 实现的信号量比 `sync.WaitGroup` 更适合"持续消费"场景，比 `semaphore.Weighted` 更轻量。

**Agent Backend 接口模式**：5 种 Agent 有 5 种完全不同的通信协议（JSON 流、JSON-RPC、stdio 等），但对外统一为 `Backend.Execute()` 接口。新增 Agent 支持只需实现接口，核心调度逻辑完全不变。这是 Go 接口驱动设计的典型应用。

**双态作者系统**：`author_type` + `author_id` 的多态设计让人类和 Agent 在数据模型中完全对等。这不是技术炫技——它让整个系统的代码路径统一，前端不需要区分"这是人说的还是 Agent 说的"。

Bohan Jiang 是 Devv.AI（AI 开发者搜索引擎）的联合创始人。他把搜索引擎领域对"信息组织"的理解带到了项目管理领域——Agent 不是信息处理的终点，而是协作流程的参与者。

一周涨 12k Star 说明开发者对"Agent 即队友"的理念有强烈共鸣。如果你正在用多个 AI Agent 做开发，Multica 的自托管方案值得尝试——特别是它的 Skills 系统，能把团队的最佳实践沉淀为 Agent 可复用的技能。

https://github.com/multica-ai/multica/
