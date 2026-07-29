# 讲透 LangGraph：从状态图到 Agent 工程化 03｜从最小例子开始：StateGraph 是怎么建图的

> 来源：小张学AI Agent (zhanglongyanmany) · 2026-07-09
> 原文：https://mp.weixin.qq.com/s/SoBgYGhP6c-jM37oJ9aOoA
> 分类：AI Agent 框架与工程化

---

> **StateGraph 是一张围绕共享状态流动的有向图。节点读取状态，返回状态的局部更新。**

---

## 一、最小 StateGraph

```python
from typing_extensions import TypedDict
from langgraph.graph import END, START, StateGraph

class State(TypedDict):
    count: int

def add_one(state: State) -> dict:
    return {"count": state["count"] + 1}

builder = StateGraph(State)
builder.add_node("add_one", add_one)
builder.add_edge(START, "add_one")
builder.add_edge("add_one", END)

graph = builder.compile()

result = graph.invoke({"count": 1})
print(result)  # {"count": 2}
```

核心概念已出现：`State`（共享状态结构）、`add_one`（读写状态的节点）、`START`/`END`（图边界）、`builder`（建图定义器）、`graph`（编译后可执行图）。

---

## 二、`State`：整张图共享的工作台

`State` 不是普通变量，而是整张图的状态 schema——共享工作台：
- 每个节点都可以读取工作台上的数据
- 节点不需要返回完整工作台，只需返回想更新的字段

复杂 Agent 的 State 可能变成：
```python
class AgentState(TypedDict):
    messages: list
    current_step: str
    tool_result: dict
    final_answer: str
```

这是整个 Agent 在多轮推理、多工具调用、多节点跳转中的"共享记忆面板"。

---

## 三、节点：签名是 `State -> Partial[State]`

LangGraph 对节点的核心期待：
```
State -> Partial[State]
```
- 输入是当前状态
- 输出是对状态的一部分更新（状态补丁）

**不要**把节点返回值理解成"函数结果"。更准确的说法：**节点返回的是一份状态补丁。**

这对后续复杂场景至关重要——多个节点追加消息、多个分支写入同一 key 时，框架需要知道更新如何合并。

---

## 四、`START` 和 `END`：图的边界标记

它们不是业务逻辑节点，而是：
- `START`：图从哪里进入
- `END`：图在哪里结束

```python
builder.add_edge(START, "add_one")  # 运行开始后进入 add_one
builder.add_edge("add_one", END)    # add_one 执行完后结束
```

等价写法（语义化）：
```python
builder.set_entry_point("add_one")
builder.set_finish_point("add_one")
```

建议入门时显式写 `START`/`END`，强迫用"图"的方式看程序。

---

## 五、Builder 阶段：登记图定义

`StateGraph` 是 builder class，负责收集图的定义，本身不是最终执行器。

初始化时准备核心容器：`nodes`（节点定义）、`edges`（普通边）、`branches`（条件边）、`schemas`（状态/输入/输出 schema）、`channels`（状态字段对应通道）、`managed`（托管值）。

**Builder = 图纸编辑器**。加节点、连线、声明入口出口，但这张图纸还不能直接跑。

---

## 六、`compile()`：图纸变可执行图

```python
graph = builder.compile()
```

`compile()` 做四件事：
1. 校验图定义是否合法
2. 准备输出通道和流式通道
3. 把节点、边、分支挂到编译后的图上
4. 返回可执行的 `CompiledStateGraph`

编译后的 `graph` 才支持：`.invoke()`, `.stream()`, `.ainvoke()`, `.astream()`

**`StateGraph` 是施工图，`CompiledStateGraph` 是可以真正开工的系统。**

---

## 七、`invoke()`：输入状态，拿到新状态

```python
result = graph.invoke({"count": 1})
```

执行过程：
1. 输入状态进入 `START`
2. 图根据边进入 `add_one`
3. `add_one` 读取 `{"count": 1}`
4. 节点返回 `{"count": 2}`
5. 框架把更新写回状态
6. 图走到 `END`
7. 返回最终状态

返回的是**最终状态**，不是节点的裸返回值。当 State 有多个字段时区别明显——节点只返回 `{"count": 2}` 表达"只更新 count，其他字段仍属图的状态管理范围"。

---

## 八、设计哲学

最小例子展示的不是"加一"，而是 LangGraph 的通用执行模型。当节点变多、边变条件、状态变复杂时，普通链式调用会越来越难维护。`StateGraph` 把问题压缩成一个统一模型：

```
状态 + 节点 + 边 + 编译 + 运行
```

---

## 九、容易混淆的点

1. **`StateGraph` 不是最终运行器**——必须先 `compile()` 再 `invoke()`
2. **节点不要返回裸值**——应该返回 dict 表示状态更新
3. **`START`/`END` 是图边界**——不是业务节点，不应用作普通节点名
4. **入门时显式写边**——哪怕只有一个节点，也写清楚 `add_edge(START, ...)` 和 `add_edge(..., END)`
5. **`Partial[State]` 是理解 reducer 的前提**——多个节点写同一 key 时，是覆盖、追加还是聚合？

---

## 核心规则

```
节点读取 State，返回 Partial[State]
START -> add_one -> END
```

---

**系列文章：**
- [01 为什么需要 LangGraph](https://mp.weixin.qq.com/s?__biz=MzYzMzM3MDQ5Ng==&mid=2247484154&idx=1&sn=01ed7a812a6843c629d90f9567a87c59&scene=21#wechat_redirect)
- [02 项目总览：monorepo、核心库和依赖关系](https://mp.weixin.qq.com/s?__biz=MzYzMzM3MDQ5Ng==&mid=2247484160&idx=1&sn=a2bb1951a43717e842e971f1dd9bd3c0&scene=21#wechat_redirect)

**源码**：https://github.com/langchain-ai/langgraph
**官方文档**：https://docs.langchain.com/oss/python/langgraph/overview
