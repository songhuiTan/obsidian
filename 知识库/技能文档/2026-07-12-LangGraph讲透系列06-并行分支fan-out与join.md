# 讲透 LangGraph：从状态图到 Agent 工程化 06｜并行分支：fan-out 与 join

> 作者：zhanglongyanmany（小张学AI Agent）
> 来源：微信公众号「小张学AI Agent」
> 日期：2026-07-12 21:31
> 系列：讲透 LangGraph 第 06 篇
> 原文：[讲透 LangGraph 06｜并行分支：fan-out 与 join](https://mp.weixin.qq.com/s/70-_l2sqw8JoBMRPs3y7NQ)

---

## 核心概念

当 Agent 需要同时做多件事（如查网页 + 查内部文档 + 查历史对话），LangGraph 用两个动作支持：

- **fan-out**：展开多个并行分支
- **join**：等待多个分支完成后再继续

对应 API：
```python
builder.add_conditional_edges(...)           # fan-out（一次返回多个目标）
builder.add_edge(["a", "b"], "join_node")     # join（等待多个上游完成）
```

---

## 一、静态 fan-out 示例

### 状态设计（必须有 reducer）

```python
class State(TypedDict):
    question: str
    docs: Annotated[list[str], operator.add]  # 多个分支都写 docs，需 reducer
    answer: str
```

### 建图

```python
builder = StateGraph(State)
builder.add_node("search_web", search_web)
builder.add_node("search_docs", search_docs)
builder.add_node("summarize", summarize)

builder.add_edge(START, "search_web")          # fan-out 1
builder.add_edge(START, "search_docs")         # fan-out 2
builder.add_edge(["search_web", "search_docs"], "summarize")  # join
builder.add_edge("summarize", END)
```

---

## 二、fan-out 与 join 是两件不同的事

| 动作 | 解决的问题 | API |
|------|-----------|-----|
| **fan-out** | 怎么让多个分支开始跑？ | 多条 `add_edge(START, node)` 或条件边返回多个目标 |
| **join** | 怎么等多个分支都跑完再继续？ | `add_edge([nodes...], target)` |

### 并行分支三段式

```
[fan-out] → [state merge (reducer)] → [join (barrier)]
```

并行分支不只是靠多条边，还需要**状态合并**和**等待汇合**。

---

## 三、源码实现：waiting edge

### 编译前 builder 阶段

`add_edge` 接受字符串或字符串列表：

```python
def add_edge(self, start_key: str | list[str], end_key: str)
```

- 单个字符串 → 普通边：`edges.add(("a", "b"))`
- 列表 → 等待边：`waiting_edges.add((("a", "b"), "join"))`

### 编译后执行阶段

等待边变成 `NamedBarrierValue`（"点名表"）：

```
预期名单：{"search_web", "search_docs"}
已到名单：seen {}
```

- 每个上游节点完成时往 barrier 写入自己的名字
- 当 `seen == 预期名单` 时触发下游节点

---

## 四、join 不负责合并数据

> join 负责等节点，reducer 负责合并数据。

```python
docs: Annotated[list[str], operator.add]  # 数据合并
builder.add_edge(["search_web", "search_docs"], "summarize")  # 执行时机
```

两者缺一不可：

| 问题 | 结果 |
|------|------|
| 只写 join 不写 reducer | 多个分支写同字段可能冲突 |
| 只写 reducer 不写 join | 下游节点无法表达"等所有分支完成再执行" |

---

## 五、条件边也可以 fan-out

```python
def route_retrieval(state: State) -> list[str]:
    return ["search_web", "search_docs"]

builder.add_conditional_edges("classify", route_retrieval)
builder.add_edge(["search_web", "search_docs"], "summarize")
```

常见 Agent 形态：**分类 → 同时多路检索 → 等待汇合 → 综合回答**

---

## 六、动态 fan-out：Send

当分支数量不固定（如用户传入 N 个主题），用 **Send**：

```python
from langgraph.types import Send

def continue_to_jokes(state):
    return [
        Send("generate_joke", {"subject": s})
        for s in state["subjects"]
    ]
```

| 场景 | 方式 |
|------|------|
| 固定多个分支 | 返回多个节点名 |
| 动态多个任务 | 返回多个 Send |

适用于 map-reduce 模式：map（多 item 并行执行同节点）→ reduce（reducer 合并回主状态）。

---

## 七、stream 顺序 ≠ 逻辑依赖顺序

- **stream** 展示实际完成顺序（谁先算完谁先输出）
- **join** 表达逻辑依赖关系（等所有上游再执行下游）

调试复杂 Agent 时务必分清。

---

## 八、工程建议

1. **分支输出要设计成可合并**——多分支写同字段需提前写 reducer
2. **join 节点只做汇总**——不要偷偷再调检索
3. **不要把所有分支接到一个巨大节点里**——复杂逻辑拆子流程
4. **固定分支用节点名，动态任务用 Send**——不要混着写
5. **复杂 fan-out 后面最好显式写 join**——方便读图人理解意图

---

## 系列拼图

到第 6 篇为止，LangGraph 关键基础已完整：

```
StateGraph          → 定义状态图
node                → 读取状态，返回状态更新
reducer             → 合并多个分支写入的状态
conditional edge    → 运行时选择下一步
fan-out             → 展开多个分支
join                → 等待多个分支汇合
```

**三句话总结：fan-out 负责展开分支，reducer 负责合并数据，join 负责等待完成。**

---

## 系列位置

| 篇目 | 主题 |
|------|------|
| 01 | 为什么需要 LangGraph：链式调用不够用 |
| 02 | 项目总览：monorepo、核心库和依赖关系 |
| 03 | 从最小例子开始：StateGraph 是怎么建图的 |
| 04 | 状态合并：Annotated 与 reducer |
| 05 | 条件边：让图自己选择下一步 |
| **06（本篇）** | **并行分支：fan-out 与 join** |

---

## 归档信息

- 公众号：小张学AI Agent
- 归档日期：2026-07-12
- 原文链接：https://mp.weixin.qq.com/s/70-_l2sqw8JoBMRPs3y7NQ
