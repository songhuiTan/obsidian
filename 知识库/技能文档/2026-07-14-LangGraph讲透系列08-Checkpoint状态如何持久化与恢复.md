# 讲透 LangGraph：从状态图到 Agent 工程化 08｜Checkpoint：状态如何持久化与恢复

> 来源：小张学AI Agent（微信公众号）
> 日期：2026-07-14
> 链接：https://mp.weixin.qq.com/s/DX3ksQxWmRxmlvbLrlFtlw

---

LangGraph 系列第 8 篇，深入 checkpoint 机制——不只是把 state 存起来，而是保存整个执行现场以便精确恢复。

## 核心要点

### 为什么只保存 state 还不够？

恢复执行至少需要两类信息：
- **数据问题**：当前各个 state channel 的值是什么？
- **调度问题**：哪些 channel 更新过，哪些节点已经看过这些更新？

Checkpoint 同时保存这两层。

### 三个关键概念

| 概念 | 表示 | 标识 |
|------|------|------|
| thread | 一条持续演进的状态历史 | `thread_id` |
| run | 对图的一次 invoke 或 stream 执行 | `run_id` |
| checkpoint | 某个 super-step 边界上的状态快照 | `checkpoint_id` |

### Checkpoint 内部保存了什么？

```python
Checkpoint(
    v=...,               # 数据格式版本
    id=...,              # 唯一标识
    ts=...,              # 创建时间
    channel_values=...,  # 各通道在该时刻的值（≈业务 state）
    channel_versions=...,# 通道当前版本号（调度时钟）
    versions_seen=...,   # 各节点已看过哪些通道版本（消费进度）
    updated_channels=...,# 本轮哪些通道更新过
)
```

核心：**状态值 + 通道时钟 + 节点消费进度**。

### BaseCheckpointSaver 五个核心接口

1. **put**：保存完整 checkpoint（super-step 边界）
2. **put_writes**：保存单个任务产生的中间写入（pending writes 容错）
3. **get_tuple**：读取指定 thread 最新或指定 checkpoint
4. **list**：按条件列出 checkpoint
5. **delete_thread**：删除整个 thread 的 checkpoint

### pending writes 的工程价值

并行任务中，成功的任务先写入 pending writes。恢复时复用它，只重试失败的任务：

```
重跑 A + 重跑 B   →   复用 A + 重试 B
```

### 三种 durability

| 模式 | 保存时机 | 适用场景 |
|------|----------|----------|
| `exit` | 图退出时 | 一次性、可重算任务 |
| `async` | 下一步运行时异步保存上一步 | 大多数 Agent 工作流（默认） |
| `sync` | 下一步开始前同步保存 | 订单、审批、昂贵任务 |

### Checkpoint vs Store

- **Checkpointer**：thread-scoped，保存图执行状态和短期线程记忆
- **Store**：cross-thread，保存长期、跨线程数据

### 六条工程建议

1. `thread_id` 要稳定且避免碰撞
2. 生产环境不要用 `InMemorySaver`
3. state 中只保存可序列化、可迁移的数据
4. 外部副作用必须幂等
5. 提前设计保留策略
6. 按业务风险选择 durability

### 恢复链路

```
get_tuple → 恢复 channel → 读取 pending writes → 根据版本准备任务 → 继续 Pregel loop
```

## 思考

Checkpoint 不是"定时保存一下 state"。它是 LangGraph 把一次易失的函数调用变成**可持续执行工作流**的基础设施。配合幂等设计，可以实现接近 exactly-once 的生产级恢复。

## 系列目录

- [01｜为什么需要 LangGraph](https://mp.weixin.qq.com/s?__biz=MzYzMzM3MDQ5Ng==&mid=2247484154&idx=1&sn=01ed7a812a6843c629d90f9567a87c59&scene=21#wechat_redirect)
- [02｜项目总览：monorepo、核心库和依赖关系](https://mp.weixin.qq.com/s?__biz=MzYzMzM3MDQ5Ng==&mid=2247484160&idx=1&sn=a2bb1951a43717e842e971f1dd9bd3c0&scene=21#wechat_redirect)
- [03｜从最小例子开始：StateGraph 是怎么建图的](https://mp.weixin.qq.com/s?__biz=MzYzMzM3MDQ5Ng==&mid=2247484183&idx=1&sn=1917c95fc16b27fb6b65ea0365f49097&scene=21#wechat_redirect)
- [04｜状态合并：Annotated 与 reducer](https://mp.weixin.qq.com/s?__biz=MzYzMzM3MDQ5Ng==&mid=2247484196&idx=1&sn=9e27cd02d7d3b45e3552f4c53a4e40e3&scene=21#wechat_redirect)
- [05｜条件边：让图自己选择下一步](https://mp.weixin.qq.com/s?__biz=MzYzMzM3MDQ5Ng==&mid=2247484208&idx=1&sn=b8365156592435e943b961a98716b564&scene=21#wechat_redirect)
- [06｜并行分支：fan-out 与 join](https://mp.weixin.qq.com/s?__biz=MzYzMzM3MDQ5Ng==&mid=2247484219&idx=1&sn=09e6c3f92bccf47fb54c6f3e9fe29414&scene=21#wechat_redirect)
- [07｜Send：动态 fan-out 与 map-reduce](https://mp.weixin.qq.com/s?__biz=MzYzMzM3MDQ5Ng==&mid=2247484240&idx=1&sn=8db8eed8fe1c92bd06bbf772f777c142&scene=21#wechat_redirect)
