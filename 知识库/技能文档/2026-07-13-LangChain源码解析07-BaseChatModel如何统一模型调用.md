---
source: "zhanglongyanmany / 小张学AI Agent"
url: "https://mp.weixin.qq.com/s/hHbN-NPmvdDAPLjsscdWCA"
date: 2026-07-13
tags: LangChain, BaseChatModel, 源码解析, 模型调用, 统一协议
---

# LangChain源码解析07：BaseChatModel如何统一模型调用

> 第七篇进入模型调用核心：BaseChatModel 怎样把 invoke、generate、stream、cache、rate limiter、callback/tracing 和 provider 差异收束成一条稳定协议。

## 一、BaseChatModel 的角色：聊天模型的统一执行协议

从源码结构看，`BaseChatModel` 的 public 方法分两类：

**命令式调用**：`invoke`、`ainvoke`、`stream`、`astream`、`batch`、`abatch`——"现在调用一次模型，怎么拿到结果"。

**声明式包装**：`bind_tools`、`with_structured_output`、`with_retry`、`with_fallbacks`、`configurable_fields`——基于当前模型生成带额外行为的新 Runnable。

`BaseChatModel` 继承 `BaseLanguageModel[AIMessage]`，又处在 Runnable 体系里：既是语言模型抽象，也是 LCEL 链路里的一个节点。具体 provider 子类（`ChatOpenAI`、`ChatAnthropic`）主要负责把标准消息、参数和工具 schema 翻译成 provider API 能懂的格式。

## 二、输入归一化：所有入口先变 PromptValue

`_convert_input` 是输入转换函数：

- 字符串 → `StringPromptValue`
- message 列表 → `ChatPromptValue`
- 已是 `PromptValue` → 直接保留

`invoke` 本身很薄——它先 `ensure_config(config)`，再调用 `generate_prompt`，最后取 `generations[0][0].message`。主干在 `generate_prompt` 和 `generate`。

## 三、generate 才是主干

`generate` 接收 `list[list[BaseMessage]]`（一批消息列表），在这里做：

1. 计算 invocation params 和 LangSmith metadata
2. 配置 callback manager，声明成一次 chat model run
3. 触发 `on_chat_model_start`，batch size、序列化模型、输入消息和参数交给 tracing
4. 对每组 messages 调用 `_generate_with_cache`
5. 合并 provider 返回的 `llm_output`
6. 成功时 `on_llm_end`，失败时 `on_llm_error`

异步 `agenerate` 保持同样结构，用 `asyncio.gather` 并发调用 `_agenerate_with_cache`。

## 四、_generate_with_cache：真正的热路径编排点

该方法实际执行顺序：

1. **Cache 判定**：先查 cache 能否直接拿结果（cache key 去掉 message id，避免同内容错过命中；命中后 `usage_metadata.total_cost` 置为 0）
2. **Rate limiter**：cache 未命中才拿 rate limiter token（cache lookup 不应被限速）
3. **V2 streaming handler**：走协议事件流
4. **普通 streaming**：走 `_stream` 并合并 chunk
5. **Provider 调用**：走子类的 `_generate`
6. **收尾**：补 message id、response metadata、写回 cache

## 五、stream 不是另一条世界线

`stream` 先问 `_should_stream`：如果子类没实现 `_stream` 或显式禁用了 streaming，退回到 `invoke` 只 yield 一次完整 `AIMessage`。

如果确定流式，打开和 `generate` 类似的 callback/tracing 生命周期，调用 `_stream`。每个 `ChatGenerationChunk` 触发 `on_llm_new_token`。流结束后 chunks 合并成 generation，触发 `on_llm_end`。

如果 provider 没给出 `chunk_position="last"`，LangChain 会补空 chunk 作为结束标记。

## 六、_should_stream：兼容 provider 差异的开关矩阵

检查条件包括：子类是否实现 `_stream`、`disable_streaming` 状态、本次是否传了 tools、显式 `stream=False`、模型实例 `streaming` 属性、callback handler 是否需要 streaming token。

最有意思的是 `disable_streaming="tool_calling"`——工具调用流式不稳定的模型可以平时 stream，带 tools 时退回非流式。

## 七、provider 子类真正要实现什么

**必需**：`_generate`（标准 messages→底层模型→`ChatResult`）、`_llm_type`

**可选增强**：`_stream`、`_agenerate`、`_astream`、`_identifying_params`、`_get_ls_params`、`_combine_llm_outputs`

子类只要做好"怎么调用这家模型"，公共生命周期由基类接管。

## 八、bind_tools 和 with_structured_output

`bind_tools` 是 provider 子类能力，把 LangChain 工具 schema 翻译成各自 provider 参数格式。

`with_structured_output` 的默认实现：先检查模型是否有 `bind_tools` → 有则将 schema 绑定为 tool 并强制 `tool_choice="any"` → 输出端接 parser（PydanticToolsParser / JsonOutputKeyToolsParser / include_raw）。

结构化输出底层复用了工具调用协议和 output parser。

## 九、结论

`BaseChatModel` 的核心价值是把"调用模型"拆成一条可组合、可追踪、可缓存、可流式、可替换的协议。向上暴露 Runnable 接口，向下只要求窄接口 `_generate`，中间统一处理 cache、rate limiter、callback、LangSmith metadata、message id、stream fallback 和结构化输出。

```python
chain = prompt | model | parser
```

这里的 `model` 是 LangChain 运行时里最重的一层协议适配器：左边接 PromptValue 和 Message，右边接 provider API，外面挂着 RunnableConfig、tracing、cache、流式事件和结构化输出。

---

**系列位置**：当前第 7 篇，前 6 篇：
1. 先看懂Agent工程骨架
2. Runnable把一切串起来
3. RunnableConfig如何追踪到底
4. Message不只是字符串
5. Tool如何从函数变成契约
6. Prompt和Parser守住两端
