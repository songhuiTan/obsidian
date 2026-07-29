# LangChain 源码解析 06：Prompt 和 Parser 守住两端

> 作者：zhanglongyanmany（小张学AI Agent）
> 来源：微信公众号「小张学AI Agent」
> 日期：2026-07-12 21:34
> 系列：LangChain 源码解析 第 06 篇
> 原文：[LangChain源码解析06：Prompt和Parser守住两端](https://mp.weixin.qq.com/s/qKk6xfZRkSCpBeQlEHBrAA)

---

## 核心主题

拆解 LangChain 中模型调用前后的两个边界：
- **输入侧（Prompt）**：业务变量 → 提示词
- **输出侧（OutputParser）**：自然语言输出 → 程序可消费的数据

两者都是 Runnable，都可进入 LCEL，都能被 tracing 记录。

---

## 一、Prompt 是输入契约，不是字符串拼接

`BasePromptTemplate` 的核心字段：

| 字段 | 说明 |
|------|------|
| `input_variables` | 调用者必须传入的变量 |
| `optional_variables` | 可选变量（常见于历史消息占位符） |
| `partial_variables` | 提前固定或延迟计算的变量 |
| `input_types` | 变量类型信息，用于生成输入 schema |
| `metadata` / `tags` | tracing 运行上下文 |

`invoke()` 会先做输入校验，缺变量时明确报错。字面量 `{foo}` 需用双花括号转义。

---

## 二、PromptValue：统一输出抽象

Prompt 格式化后返回 `PromptValue`，核心方法：
- `to_string()` → 纯字符串
- `to_messages()` → 消息列表

解决模型类型差异：纯文本模型要字符串，聊天模型要消息列表。上层链路不需要每处判断传什么格式。

- **StringPromptValue** → 字符串 或 HumanMessage
- **ChatPromptValue** → 一组消息，可转 messages 或 buffer string

---

## 三、PromptTemplate 三种格式

| 格式 | 态度 | 特点 |
|------|------|------|
| `f-string` | **默认推荐** | 禁止属性/索引访问、纯数字变量名、嵌套替换；保持变量可静态识别校验 |
| `mustache` | 支持 | 可为嵌套变量生成 Pydantic 输入 schema；访问 Python 对象属性时被拦住 |
| `jinja2` | 可用但警告 | 反复强调不要接收不可信模板，sandbox 只是 best-effort |

---

## 四、ChatPromptTemplate：消息组合器

支持多种 message-like 表达：
- 已构造好的 `BaseMessage`
- 消息模板
- `('human', '{input}')` 二元组
- 单个字符串

**MessagesPlaceholder** 是关键：表示"这里不是一段文本，而是一组已经存在的消息"。用 `convert_to_messages` 统一处理 tuple/字符串/Message 实例，支持 `n_messages` 截取最近若干条历史。

---

## 五、Parser 也是 Runnable

`BaseOutputParser` 实现 Runnable 接口，输入可以是字符串或 `BaseMessage`：
- 传入消息 → 包装成 `ChatGeneration`
- 传入字符串 → 包装成 `Generation`

默认 `parse_result()` 只取第一个 generation，再交给 `parse(text)`。

两个重要接口：
- `get_format_instructions()`：输出格式要求反馈给 Prompt
- `parse_with_prompt()`：解析失败时拿到原始 prompt 作为修复上下文

---

## 六、三层输出收束

| Parser | 能力 | 流式 |
|--------|------|------|
| `StrOutputParser` | 抽成普通字符串 | ✅ 逐块输出文本 |
| `JsonOutputParser` | 解析 JSON，能从 Markdown code block 提取 | ✅ partial JSON（能解析多少先产多少），`diff=True` 输出 JSONPatch |
| `PydanticOutputParser` | JSON → Pydantic model 校验（v1/v2 兼容） | ❌ 失败时携带目标模型名、原始 completion 和校验错误 |

Parser 不只是事后解析，也会把 schema 转成 format instructions 提前放进 Prompt，让模型尽量按目标结构输出。

---

## 七、ToolsParser：与 Tool 体系接续

解析 `AIMessage.tool_calls` 或 provider 原始 tool call payload：
- `JsonOutputToolsParser` → name/type/args/id 结构
- `PydanticToolsParser` → 根据工具名找到对应 Pydantic model，校验 args

未知工具名抛 `OutputParserException`，列出可用工具。

与第 5 篇 Tool 体系是同一战线两端：模型输出 tool call → parser 变结构化工具请求 → 运行时执行 → 回填 `ToolMessage`。

---

## 八、StructuredPrompt：输入输出合一

继承 `ChatPromptTemplate`，额外携带结构化输出 schema。当通过 `|` 接到语言模型时，自动变成：
```python
prompt | model.with_structured_output(schema)
```

LangChain 方向：输入 Prompt、模型调用、输出结构不是三个独立工具，而是一条可声明、组合、追踪的契约链。

---

## 核心结论

> Prompt 侧负责变量推断、输入 schema、partial 绑定、消息模板、多模态模板和历史消息插入；
> Parser 侧负责把候选生成结果解析成字符串、JSON、Pydantic 对象或 tool call，并在流式场景支持部分解析。

`prompt | model | parser` 不只是 LCEL 写法，而是 LangChain 工程方法论主线：**每一步都声明输入输出，每一步都能被组合、追踪和替换。**

---

## 系列位置

| 篇目 | 主题 |
|------|------|
| 第 1 篇 | Agent 工程骨架 |
| 第 2 篇 | Runnable 把一切串起来 |
| 第 3 篇 | RunnableConfig 如何追踪到底 |
| 第 4 篇 | Message 不只是字符串 |
| 第 5 篇 | Tool 如何从函数变成契约 |
| **第 6 篇（本篇）** | **Prompt 和 Parser 守住两端** |

源码参考：https://github.com/langchain-ai/langchain

---

## 归档信息

- 公众号：小张学AI Agent
- 归档日期：2026-07-12
- 原文链接：https://mp.weixin.qq.com/s/qKk6xfZRkSCpBeQlEHBrAA
