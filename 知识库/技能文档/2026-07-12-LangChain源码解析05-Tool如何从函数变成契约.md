# LangChain 源码解析 05：Tool 如何从函数变成契约

> 作者：zhanglongyanmany（小张学AI Agent）
> 来源：微信公众号「小张学AI Agent」
> 日期：2026-07-12 21:29
> 系列：LangChain 源码解析 第 05 篇
> 原文：[LangChain源码解析05：Tool如何从函数变成契约](https://mp.weixin.qq.com/s/RdojltI3OiONkSsG0rTTaA)

---

## 核心问题

怎样把一个普通 Python 函数变成**模型能理解、运行时能校验、Agent 能追踪、结果能回填到消息列表**的工程契约。

Tool 同时服务三方：
- **模型**：需要知道有哪些工具、叫什么、什么时候用、参数结构
- **运行时**：需要校验参数、注入系统上下文、执行函数
- **消息链路**：需要把结果包装成 `ToolMessage`，用 `tool_call_id` 对回 `AIMessage.tool_calls`

---

## 一、BaseTool：不只是 callable

`BaseTool` 继承自 Runnable，把工具变成可 `invoke` / `ainvoke` 的组件，额外维护：

- `name`、`description`
- `args_schema`、`tool_call_schema`
- `response_format`
- 错误处理和 callback 信息

不能只是 callable — 只保存函数对象模型不知道怎么调；只保存 schema 运行时不知道怎么执行；只执行函数 Agent 无法把结果放回 Message 闭环。

---

## 二、@tool 装饰器

```python
from langchain.tools import tool

@tool(parse_docstring=True)
def search_docs(query: str, limit: int = 5) -> str:
    """Search internal documents.

    Args:
        query: Search query.
        limit: Maximum number of results.
    """
    return "..."
```

背后：函数签名 → 参数 schema，docstring 摘要 → 工具描述，Args 说明 → 字段描述。模型看到的不是 Python 函数，而是一份**结构化工具说明**。

---

## 三、create_schema_from_function

签名 → Pydantic schema 的核心路径，四步：

1. 读取 `inspect.signature(func)`，确认参数列表
2. 用 Pydantic 参数校验生成临时模型
3. 过滤框架参数（`run_manager`、`callbacks`、RunnableConfig 对应参数）
4. 把 docstring 或 `Annotated` 的描述补进字段描述

开启 `parse_docstring=True` 时按 Google-style 解析参数说明，对不上时报错。

---

## 四、两个 schema 分层

| Schema | 用途 | 内容 |
|--------|------|------|
| `get_input_schema()` | 运行时输入校验 | 包含全部参数（含注入参数） |
| `tool_call_schema` | 给模型看的 schema | 仅非注入字段，过滤运行时参数 |

**关键分层**：模型只需要知道业务参数（如 `query`、`limit`），不需要知道 `state`、`store`、`runtime` 等系统上下文。源码对 `tool_call_schema` 做了 memo 和 JSON schema 缓存，避免热路径重复构造。

---

## 五、InjectedToolArg：模型不该填的参数

```python
from langchain.tools import ToolRuntime, tool
from langchain_core.messages import ToolMessage
from langchain_core.tools import InjectedToolCallId

@tool
def save_note(
    note: str,
    runtime: ToolRuntime,
    tool_call_id: Annotated[str, InjectedToolCallId],
) -> ToolMessage:
    """Save a note for the current session."""
    user_id = runtime.context.get("user_id", "anonymous")
    return ToolMessage(
        content=f"saved for {user_id}: {note}",
        tool_call_id=tool_call_id,
    )
```

- 模型只看到 `note`
- `runtime` 来自系统
- `tool_call_id` 来自模型上一轮 ToolCall 外层 ID（**不支持模型在 args 里伪造**）

工具参数校验失败时，错误消息只包含模型可修正的参数问题，不暴露 `state`、`store`、`runtime` 或敏感值。

---

## 六、执行路径：ToolCall → ToolMessage

```
ToolCall(type='tool_call', name, args, id)
  → _prep_run_args() 取出 args + 外层 id
  → _parse_input() 根据 args_schema 校验 + 默认值
  → _to_args_and_kwargs() 转成位置/关键字参数
  → _run() / _arun() 执行
  → _format_output() 包装成 ToolMessage（含 name, tool_call_id, status）
```

`response_format='content_and_artifact'`：返回 `(content, artifact)` 二元组，`content` 给模型看，`artifact` 保存完整原始产物。

---

## 七、错误处理

`BaseTool` 提供两类入口：
- `handle_validation_error` — 参数校验错误
- `handle_tool_error` — 工具主动抛出的 `ToolException`

配置为 `True`、字符串或 callable 时，错误转为工具输出 → `status='error'` 的 `ToolMessage`，回到消息列表，给 Agent 留修复空间。

---

## 八、Provider Adapter：统一 Tool，翻译给不同模型

LangChain 先维护统一的 `BaseTool` 语义，再由 adapter 翻译：

- **OpenAI**：`convert_to_openai_tool` / `convert_to_openai_function`
- **Anthropic**：转成 Anthropic 需要的格式

应用代码面向 `BaseTool`、`@tool`、Python 类型系统编程；模型 API 细节交给集成包。

---

## 核心结论

> Tool 体系的核心，不是让你少写几行函数包装代码，而是把**"模型想调用能力"和"程序真实执行能力"之间的边界做清楚。**

用 `@tool` 降低声明成本 → `create_schema_from_function` 生成参数契约 → `tool_call_schema` 隔离模型可见字段 → 注入参数承载系统上下文 → `_format_output` 放回 Message 体系。

**模型只负责提出结构化请求，运行时负责把请求安全地变成真实动作。**

---

## 系列位置

| 篇目 | 主题 |
|------|------|
| 第 1 篇 | Agent 工程骨架 |
| 第 2 篇 | Runnable 把一切串起来 |
| 第 3 篇 | RunnableConfig 如何追踪到底 |
| 第 4 篇 | Message 不只是字符串 |
| **第 5 篇（本篇）** | **Tool 如何从函数变成契约** |
| 第 6 篇 | Prompt 和 Parser 守住两端 |

---

## 归档信息

- 公众号：小张学AI Agent
- 归档日期：2026-07-12
- 原文链接：https://mp.weixin.qq.com/s/RdojltI3OiONkSsG0rTTaA
