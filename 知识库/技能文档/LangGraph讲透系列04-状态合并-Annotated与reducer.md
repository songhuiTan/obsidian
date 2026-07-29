# 讲透 LangGraph：从状态图到 Agent 工程化 04｜状态合并：Annotated 与 reducer

> **来源**：微信公众号「小张学AI Agent」(zhanglongyanmany)
> **日期**：2026-07-10
> **链接**：https://mp.weixin.qq.com/s/ql6BoaFu1ygLtNmGReOoyQ
> **标签**：`LangGraph` `StateGraph` `reducer` `Annotated` `状态合并` `Agent工程化`
> **系列**：LangGraph 讲透系列 04/?

---

## 核心问题

如果两个节点都返回同一个字段，LangGraph 怎么处理？

```python
return {"messages": [...]}
```

**答案取决于字段背后的 channel 类型。**

---

## 一、默认通道：`LastValue`

当写 `class State(TypedDict): answer: str` 时，LangGraph 会把 `answer` 变成 `LastValue` 通道：

- 这一轮没有更新 → 不改
- 这一轮有一个更新 → 接收
- 这一轮有多个更新 → **报错**

这是安全设计：框架不知道你的业务语义（选第一个？最后一个？拼接？列表？），因此默认保守。

---

## 二、用 `Annotated` 声明 reducer

```python
from typing import Annotated
import operator
from typing_extensions import TypedDict

class State(TypedDict):
    items: Annotated[list[str], operator.add]
```

含义：`items` 的值为 `list[str]`，收到多个更新时用 `operator.add`（即列表拼接）合并。

```python
# 节点可以并行写
def node_a(state: State) -> dict:
    return {"items": ["a"]}

def node_b(state: State) -> dict:
    return {"items": ["b"]}

# 结果: [] -> ["a"] -> ["a", "b"]
```

---

## 三、源码识别机制

`StateGraph(State)` 初始化路径：

```
StateGraph(State) → _add_schema → _get_channels → _get_channel(name, annotation)
```

`_get_channel()` 判断顺序：
1. 是否是 managed value
2. 是否直接声明了 channel
3. 是否是 `Annotated[..., reducer]`
4. 否则用 `LastValue`

`Annotated` 被识别为 reducer 的条件：
- metadata 最后一项是 **callable**
- 该 callable 有 **两个位置参数** → 签名 `(Value, Value) -> Value`

满足条件后包装为 `BinaryOperatorAggregate` 通道。

---

## 四、`BinaryOperatorAggregate` 工作原理

每次收到 updates，按 reducer 把新值合进去：

```python
items: Annotated[list[str], operator.add]

# 初始: []
# 节点A: [] + ["a"] -> ["a"]
# 节点B: ["a"] + ["b"] -> ["a", "b"]
```

**reducer 设计原则**：
- 同类型输入输出
- 合并结果稳定
- 不修改入参
- 对并发分支的顺序不敏感

---

## 五、messages 的特殊处理

不要直接用 `operator.add` 处理消息列表：

```python
# ❌ 机械拼接
messages: Annotated[list, operator.add]

# ✅ 带消息语义的合并
from langgraph.graph import add_messages

class State(TypedDict):
    messages: Annotated[list, add_messages]
```

`add_messages` 能力：
- 默认 append-only
- **ID 相同则替换**（而非追加）
- 支持删除消息
- 转换多种消息表示为 LangChain message 对象

LangGraph 提供了内置状态 `MessagesState`：

```python
class MessagesState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
```

---

## 六、何时用普通字段 vs reducer

**判断标准：这个字段在一个 step 里是否可能收到多个写入？**

| 场景 | 推荐 | 示例 |
|------|------|------|
| "当前值"型 | 普通字段 `LastValue` | `final_answer`, `current_route` |
| "累积型" | `Annotated` + reducer | `messages`, `steps`, `tool_results` |

```python
class State(TypedDict):
    messages: Annotated[list, add_messages]
    steps: Annotated[list[str], operator.add]
    tool_results: Annotated[list[dict], operator.add]
    token_count: Annotated[int, operator.add]
```

---

## 七、逃生口：`Overwrite`

已声明 reducer 的字段也可以整体替换（绕过 reducer）。谨慎使用，易让读者困惑。

---

## 小结

```
普通字段默认是 LastValue
Annotated 字段可以声明 reducer
messages 推荐使用 add_messages
```

**工程判断**：如果一个字段可能被多个节点写，就不要让框架猜，自己声明合并规则。

从这篇开始，看 `State` 不能只看字段类型，还要看有没有 `Annotated` 及其背后的 reducer。

---

## 系列文章

- [01 为什么需要 LangGraph](https://mp.weixin.qq.com/s?__biz=MzYzMzM3MDQ5Ng==&mid=2247484154&idx=1&sn=01ed7a812a6843c629d90f9567a87c59&scene=21#wechat_redirect)
- [02 项目总览](https://mp.weixin.qq.com/s?__biz=MzYzMzM3MDQ5Ng==&mid=2247484160&idx=1&sn=a2bb1951a43717e842e971f1dd9bd3c0&scene=21#wechat_redirect)
- [03 从最小例子开始](https://mp.weixin.qq.com/s?__biz=MzYzMzM3MDQ5Ng==&mid=2247484183&idx=1&sn=1917c95fc16b27fb6b65ea0365f49097&scene=21#wechat_redirect)

---

**源码**：https://github.com/langchain-ai/langgraph
**文档**：https://docs.langchain.com/oss/python/langgraph/overview
