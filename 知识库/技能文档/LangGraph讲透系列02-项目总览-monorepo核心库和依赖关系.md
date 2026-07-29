---
source: 微信公众号
title: "讲透 LangGraph：从状态图到 Agent 工程化 02｜项目总览：monorepo、核心库和依赖关系"
author: 小张学AI Agent
date: 2026-07-08
url: https://mp.weixin.qq.com/s/zAaesLAppmY21SMbdPUm3w
tags:
  - LangGraph
  - 源码分析
  - monorepo
  - StateGraph
  - Pregel
  - checkpoint
  - Agent工程化
---

# 讲透 LangGraph：从状态图到 Agent 工程化 02｜项目总览：monorepo、核心库和依赖关系

> LangGraph 1.2.8 monorepo 结构解析 + 源码阅读路线图
>
> 上一篇：[[LangGraph讲透系列01-为什么需要LangGraph]]

## 一、monorepo 结构总览

LangGraph 不是一个单包项目，而是 monorepo。一个仓库里放了多个可独立发布、独立测试、独立维护的库。

```
libs/
├── langgraph              # 核心图执行框架
├── prebuilt               # 高层 Agent 预置能力
├── checkpoint             # checkpoint 接口
├── checkpoint-sqlite      # checkpoint SQLite 实现
├── checkpoint-postgres    # checkpoint Postgres 实现
├── checkpoint-conformance # 测试一致性套件
├── cli                    # 命令行工具
├── sdk-py                 # 服务端 API SDK (Python)
└── sdk-js                 # 服务端 API SDK (JavaScript)
```

## 二、核心包：`libs/langgraph`

对应发布包 `langgraph==1.2.8`。依赖关系：

```
langgraph
├── langchain-core         # Runnable、消息、工具基础抽象
├── langgraph-checkpoint   # checkpoint/cache/store 接口
├── langgraph-sdk          # LangGraph Server/API 连接
├── langgraph-prebuilt     # 高层预置能力
├── xxhash                 # 稳定哈希
└── pydantic               # schema 校验
```

### 核心源码目录

```
langgraph/
├── graph/       # 用户建图 API，StateGraph 在这里
├── pregel/      # 图执行引擎，Pregel 在这里
├── channels/    # 状态通信与更新规则
├── stream/      # 流式输出与事件转换
├── managed/     # 托管值，如 RemainingSteps
├── _internal/   # 内部配置、序列化、重试、队列
├── runtime.py   # Runtime、context、运行控制
├── types.py     # Command、Send、Interrupt 核心类型
└── errors.py    # 错误类型
```

**三个最重要的入口：**

| 文件 | 层级 | 职责 |
|------|------|------|
| `graph/state.py` | 用户 API 层 | 用户怎么定义图 |
| `pregel/main.py` | 执行模型层 | 图怎么被执行 |
| `pregel/_loop.py` | 运行推进层 | 一次运行怎么推进、保存和恢复 |

## 三、预置能力：`libs/prebuilt`

对应 `langgraph-prebuilt`。不是核心执行引擎，而是上层模板。

关键入口：
- `create_react_agent` — 模型调用+工具调用+条件跳转的常见 Agent 组织
- `ToolNode` — 工具节点

> 它本质上还是在组装一张图。`prebuilt` 不是另一个系统，而是站在 `StateGraph` 之上的上层模板。

## 四、持久化基础：`libs/checkpoint`

对应 `langgraph-checkpoint==4.1.1`。三类抽象：

```
langgraph/
├── checkpoint/
│   ├── base/    # 基础接口
│   ├── memory/  # 内存实现
│   └── serde/   # 序列化
├── cache/
│   ├── base/
│   ├── memory/
│   └── redis/
└── store/
    ├── base/
    └── memory/
```

| 组件 | 职责 |
|------|------|
| checkpoint | 保存图执行状态 |
| cache | 缓存节点执行结果 |
| store | 长期存储和检索 |
| serde | 序列化与反序列化 |

SQLite 和 Postgres 是具体实现，都依赖 `langgraph-checkpoint` 接口。核心引擎只依赖 `BaseCheckpointSaver` 抽象接口。

## 五、CLI 和 SDK

- `libs/cli` → `langgraph-cli`：配置、dev server、deploy、docker、依赖追踪
- `libs/sdk-py` → `langgraph-sdk`：LangGraph API 交互，同步/异步双客户端
- `libs/sdk-js`：JavaScript/TypeScript SDK 指引

## 六、为什么这样拆？

LangGraph 有意识地区分四件事：

| 层次 | 模块 | 关注点 |
|------|------|--------|
| 定义图 | `langgraph/graph/` | 用户怎么表达 Agent 工作流 |
| 执行图 | `langgraph/pregel/` | 图怎么一步步推进 |
| 保存图 | `checkpoint/` | 执行过程怎么保存、恢复、回放 |
| 部署图 | `cli/` + `sdk-py/` | 图怎么变成服务，远程调用 |

## 七、源码阅读路线

**建议顺序：**

1. **`StateGraph`** — 用户怎么定义状态、节点和边
2. **`compile()`** — `StateGraph` → `CompiledStateGraph`，从定义图进入执行图的入口
3. **`Pregel`** — 执行模型：Plan、Execution、Update、stream、callback、retry
4. **checkpoint** — 理解执行过程后再看状态保存和恢复
5. **prebuilt** — 最后看 `create_react_agent` 等高层 API

**高重力文件：**
- `pregel/main.py`
- `pregel/_loop.py`
- `graph/state.py`
- `prebuilt/chat_agent_executor.py`
- `prebuilt/tool_node.py`
- `checkpoint/base/__init__.py`

## 八、总结

> 复杂 Agent 的核心是状态流转；而一个工程化框架必须同时解决定义状态、执行状态、保存状态和服务化状态。

LangGraph 的 monorepo 正是围绕这几件事拆出来的。
