---
source: 微信公众号
title: AgentScope(Java)2.0 怎么做到一个 Agent 服务所有用户？聊聊无状态引擎和 AgentState
author: AI开发学习实验室
date: 2026-06-23
url: https://mp.weixin.qq.com/s/DZZJDHykKvg-e8e2TFnINA
tags:
  - AgentScope
  - AgentState
  - 无状态引擎
  - Java
  - Harness
  - 状态管理
---

# AgentScope(Java)2.0 怎么做到一个 Agent 服务所有用户？聊聊无状态引擎和 AgentState

> AgentScope 2.0 的上下文和状态管理可以浓缩成一个核心思路：**Agent 是引擎，AgentState 是燃料**。
> 
> 来源：AI开发学习实验室
> 系列：AgentScope(Java)2.0 深度解析系列

## 核心设计：Agent 实例 ≠ 用户会话

2.0 中一个 HarnessAgent 实例只持有**不可变配置**——system prompt、模型、工具集、中间件链。所有跟用户相关的**可变数据**，全塞进一个叫 **AgentState** 的对象里，按 `(userId, sessionId)` 二元组索引。

```java
// Alice 的请求
agent.call(msg, RuntimeContext.builder()
    .userId("alice").sessionId("s1").build()).block();

// Bob 的请求——同一个 agent 实例，完全并行
agent.call(msg, RuntimeContext.builder()
    .userId("bob").sessionId("s2").build()).block();
```

不需要注册表，不需要 per-user 实例池。**Agent 是无状态的引擎，AgentState 才是会话。**

## AgentState 里装了什么

一份「当前对话的完整快照」。框架在每次 `call()` 入口从存储加载，`call()` 出口自动保存。调用方不需要自己管理这个对象。

- **getContext()**：当前对话历史——用户输入、assistant 回复、工具调用、工具结果
- **getSummary()**：如果开了上下文压缩，摘要存这里
- **getPermissionContext()**：工具权限规则
- **getPlanModeContext()**：Plan Mode 是否激活、计划文件路径
- **getTasksContext()**：todo_write 维护的任务清单
- **getToolContext()**：工具组激活状态
- **InterruptControl**：per-session 中断信号（瞬态，不序列化，`@JsonIgnore transient`）

> **设计细节：** InterruptControl 标记为 transient，永远不会写进状态存储——如果 session 故障转移到另一台机器，中断信号从零开始。而 `shutdownInterrupted` 是反过来的，会被持久化，记录 session 是否被优雅停机中断。

## 一次 call() 的完整链路

```
call(msgs, RuntimeContext(userId, sessionId))
  │
  ├─ per-session 门: 同 (uid, sid) 串行, 不同会话并行
  │
  ▼  从缓存或 stateStore 加载 AgentState
  │  注入到 RuntimeContext: rc.setAgentState(state)
  │
  ▼  推理循环
  │  中间件就地改写 state.contextMutable()
  │  (压缩、Plan、todo_write、权限调整……都在改它)
  │
  ▼  保存 AgentState
  │  stateStore.save(userId, sessionId, "agent_state", state)
  │
  ▼  返回结果
```

> **注意：** 在中间件和工具里访问 AgentState 时，要用 `RuntimeContext.resolveAgentState(ctx, agent)`，不是 `agent.getAgentState()`。因为并发场景下后者返回的是最后一次活跃 session 的状态。

## 状态存储后端

框架针对 `AgentStateStore` 接口有四个内置实现：

| 实现 | 模块 | 适用场景 |
|------|------|----------|
| InMemoryAgentStateStore | core | 单测、演示。进程退出全丢 |
| JsonFileAgentStateStore | core | 单机开发，文件落盘，不能跨节点 |
| RedisAgentStateStore | extensions-redis | 生产首选，多副本共享 |
| MysqlAgentStateStore | extensions-mysql | 需要状态沉淀进关系型库（审计、报表） |

**安全检查：** 如果用了分布式文件系统（SandboxFilesystemSpec / RemoteFilesystemSpec），同时状态存储还是单机的，`build()` 直接抛 `IllegalStateException`。

## 故障转移：跨机器恢复

只要状态存储是分布式的（如 Redis），迁移就是自动的：

```
// 节点 A：Alice 在这聊了一会儿
agentA.call(msg, RuntimeContext.builder()
    .sessionId("alice-001").userId("alice").build()).block();

// 节点 A 挂了。请求漂到节点 B
// 自动从 Redis 拉到节点 A 留下的 AgentState
agentB.call(nextMsg, RuntimeContext.builder()
    .sessionId("alice-001").userId("alice").build()).block();
```

用户感知不到节点切换。甚至跨场景也能接续：Web UI 聊到一半，切到 CLI 继续聊。

## Per-session 中断

传统 `interrupt()` 无参调用会整个 agent 停掉。2.0 的 InterruptControl 是 per-session 的：

```java
// 只中断 Alice 的这个 session，Bob 不受影响
agent.interrupt("alice", "session-001");

// 还能在中断时注入一条用户消息
agent.interrupt("alice", "session-001",
    Msg.userMsg("请停下来做个总结。"));
```

## 并发规则（三句话）

1. 不同 `(userId, sessionId)` → **完全并行**
2. 相同 `(userId, sessionId)` → per-session 异步门按 **FIFO 串行**，无需外部锁
3. `interrupt(userId, sessionId)` → **精确命中单个 session**

## AgentState vs Memory

| | AgentState（上下文） | Memory（记忆） |
|---|---|---|
| 生命周期 | 会话内 | 跨会话持久化 |
| 存什么 | 当前对话历史、摘要、权限、任务 | 长期事实、偏好、经验 |
| 谁管理 | 框架自动加载/保存 | 后台任务定期整理 |
| 存储位置 | AgentStateStore | 文件系统（MEMORY.md + 日流水账） |

AgentState 是「当前在聊什么」，Memory 是「以前聊过什么值得记住的」。每次 agent 启动推理时，MEMORY.md 的内容会注入 system prompt——这是两个系统交汇的地方。

## RuntimeContext 自定义数据

```java
RuntimeContext ctx = RuntimeContext.builder()
    .userId("alice")
    .sessionId("s-001")
    .put("request_id", "req-2026-06-20-abc")
    .put(MyTenantInfo.class, new MyTenantInfo("tenant-7"))
    .build();
```

中间件和工具通过 `ctx.get("request_id")` 或 `ctx.get(MyTenantInfo.class)` 取出来用。这些数据不持久化，每次 call 自己带。

> **注意：** AgentStateStore 后端在 builder 时绑定，不能通过 RuntimeContext per-call 切换。per-call 变化的是它寻址的 `(userId, sessionId)` 槽位，不是存储实例本身。

## 总结

一个无状态的 Agent 实例 + 按 `(userId, sessionId)` 索引的 AgentState + 可插拔的状态存储后端 = 从单机开发到多副本生产的平滑路径。不需要自己写注册表、不需要自己管锁、不需要自己处理故障转移。
