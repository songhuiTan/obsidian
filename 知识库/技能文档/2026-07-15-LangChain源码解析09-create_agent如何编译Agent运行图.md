---
title: "LangChain源码解析09：create_agent如何编译Agent运行图"
source: "小张学AI Agent"
source_url: "https://mp.weixin.qq.com/s/ViJHuCa32_AUGYn4yNKLDQ"
date: "2026-07-15"
tags: [LangChain, 源码解析, create_agent, StateGraph, Agent]
---

> 九篇进入 LangChain v1 最关键的应用入口：create_agent。

`create_agent` 不是"立即运行 Agent"的函数，而是"生成 Agent 运行时"的工厂。它把配置编译成一张 `StateGraph`。

## 核心结论

```
create_agent = 配置归一化 + 运行节点装配 + 状态图编译
```

三个关键设计结果：
1. Agent 循环从隐藏控制流变成显式图结构，路径可观察、可测试、可中断
2. 扩展点分配到正确层级：改变生命周期 → 图节点，包裹单次调用 → wrapper，新增状态 → schema
3. 模型和工具只负责窄职责，checkpointer/store/streaming/HITL 由图运行时统一承接

---

## 一、返回值是 CompiledStateGraph，不是 Agent 类

如果返回传统 `AgentExecutor`，源码会围绕 while 循环组织。但 `CompiledStateGraph` 要求在构建阶段回答三类问题：
1. 图里有哪些节点
2. 节点之间有哪些固定边和条件边
3. 哪些状态字段、运行时服务和流式转换器需要交给图执行引擎

最终得到的是一张可暂停、恢复、持久化、观察和嵌套的运行图。

## 二、第一阶段：宽松输入收束成内部对象

- **模型归一化**：字符串 → `init_chat_model()`，之后只面对统一模型协议
- **系统提示词归一化**：字符串 → `SystemMessage`，但每次模型调用前临时放入，不属于对话历史本身
- **工具归一化**：`tools=None` → 空列表；工具拆成 dict（provider built-in）和 callable/BaseTool（client-side）
- **response_format 归一化**：Pydantic/TypedDict/JSON Schema → `AutoStrategy`（运行时根据模型能力自动选择）

## 三、ToolNode 不是所有工具的容器

| 来源 | 执行位置 | 是否进 ToolNode |
|------|---------|---------------|
| 用户普通工具 / middleware 工具 | 当前进程 | ✅ |
| Provider built-in tools（搜索、代码执行等） | 模型服务端 | ❌ 只绑定给模型 |
| middleware 实现 `wrap_tool_call` 时 | 动态注册 | ✅ 即使无静态工具也会创建 |

provider 工具和客户端工具可以出现在同一份 `ModelRequest.tools` 中，却不会被错误地交给同一个执行器。

## 四、middleware 有两种形态：图节点与调用包装器

**生命周期节点**（改变图结构）：
- `before_agent` / `before_model` / `after_model` / `after_agent`
- 只要 middleware 覆盖了对应方法，`create_agent` 就注册为 `RunnableCallable` 节点

**调用包装器**（不成为独立节点）：
- `wrap_model_call` / `awrap_model_call`
- `wrap_tool_call` / `awrap_tool_call`
- 组合成嵌套 handler，包在 model 节点或 ToolNode 内部

区别：生命周期 hook 改变图的结构，wrapper 改变某个节点内部的一次调用。

## 五、AgentState：整张图共享的主干

默认三字段：
- `messages`：`add_messages` reducer 累积 Human/AI/Tool 消息
- `jump_to`：ephemeral private state，供 middleware 临时改路由
- `structured_response`：只出现在输出侧

state schema 不只是类型提示，它同时定义了通道、reducer、输入边界和输出边界。合并规则：后声明覆盖前声明，调用者显式传入的 base state 优先级最高。

## 六、model 节点不是简单的 model.invoke

每轮进入 model 节点时，先构造 `ModelRequest`（model/tools/system_message/response_format/messages/state/runtime），这是 middleware 能动态改模型、工具、提示词的关键。

真正调用前还经过一轮运行时装配：
1. 校验 middleware 动态加入的 client-side tool 是否有执行路径
2. 根据模型 profile 选择 ProviderStrategy 或 ToolStrategy
3. 合并普通工具、provider tools 和结构化输出工具
4. `bind_tools` / `bind` → 本轮 Runnable

模型结果经 `_handle_model_output` 解析 provider structured output，再统一变成图状态更新。

model 节点本质是一个"小型请求管线"：组装请求 → 执行 middleware → 绑定工具 → 调用模型 → 解析输出 → 提交状态变化。

## 七、节点注册：同步与异步同一张图

```
graph.add_node("model", RunnableCallable(model_node, amodel_node, trace=False))
```

`RunnableCallable` 同时持有同步和异步实现，运行时根据调用方式选择正确路径。图上只保留对状态和路由有独立意义的阶段。

## 八、四个生命周期位置，执行频率不同

| 位置 | 执行时机 | 执行次数 |
|------|---------|---------|
| `entry_node` | 整次 Agent 运行入口 | 1 次 |
| `loop_entry_node` | 每轮模型循环入口 | 每轮 |
| `loop_exit_node` | 每轮模型调用后的出口 | 每轮 |
| `exit_node` | 整次 Agent 运行结束前 | 1 次 |

- `before_agent` / `after_agent`：只运行一次
- `before_model` / `after_model`：每轮都运行

middleware 顺序：前置 hook 按注册顺序进入，后置 hook 反向退出（类似嵌套调用栈）。

## 九、边按能力剪裁，不是固定模板

- 最小 Agent（无工具、无结构化输出）：`START → model → END`
- 有 client-side tools：model → 条件边 → tools / END
- `return_direct=True` 或结构化输出完成 → 走向退出
- middleware 声明 `can_jump_to` → 对应节点获得条件边

边让"哪些路径可能发生"在编译时显式可见。

## 十、compile 把图变成真正的运行时

注入的能力：
- `checkpointer`：对话记忆、暂停恢复
- `store`：跨 thread 长期数据
- `interrupt_before/after`：指定节点前后暂停
- `cache`：缓存图节点执行结果
- `transformers`：ToolCall → Subagent → middleware → 调用者

## 十一、第九篇的结论

`create_agent` 看起来是一个高层便捷 API，实际承担的是编译器式工作：把声明式配置翻译成一张可执行、可恢复、可组合的状态图。

---

## 战略分析

### 与当前工作的映射

这是 LangChain 源码解析系列的第九篇，也是进入 `create_agent` 这个核心入口的深度拆解。对我们来说，有几个关键启示：

**1. 图即 Agent 运行时**

LangChain 的 `create_agent` 选择把 Agent 循环编译成 `StateGraph`，而不是写一个 while 循环。这个选择意味着：
- 暂停/恢复/持久化是图的固有属性，不是事后追加
- 节点间路由显式可见，便于调试和测试
- middleware 被精确分配到"图结构变更"或"调用包装"两个层级

Hermes Agent 目前的 Agent Loop（model + tools + memory）虽然工作，但缺少这种"运行时编译"的工程美感。如果未来需要支持复杂多步编排，LangGraph 的设计很值得借鉴。

**2. provider tool vs client-side tool 的区分**

源码对 provider built-in tools 和 client-side tools 做了严格区分——前者由模型服务端执行，后者由本地 ToolNode 执行。这层区分在 Hermes 里同样重要：FAL 图片生成、SiliconFlow ASR 等属于 provider tools，文件读写、终端命令属于 client-side tools。目前我们的 Tool 注册没有显式区分这两类，未来可以考虑引入类似策略。

**3. middleware 的分层设计**

middleware 被分成"改变图结构的 hook"和"包装单次调用的 wrapper"，后者可以组合成嵌套栈。这个设计比简单的"请求前/请求后"拦截器更精细。Hermes 的 cron/skill 体系也可以借鉴——在 cron 执行链中区分"生命周期级"和"调用级"的拦截。

### 系列位置

| 篇目 | 主题 | 归档状态 |
|------|------|---------|
| 01 | Agent 工程骨架 | 未归档 |
| 02 | Runnable 把一切串起来 | ✅ 已归档 |
| 03 | RunnableConfig 如何追踪到底 | 未归档 |
| 04 | Message 不只是字符串 | 未归档 |
| 05 | Tool 如何从函数变成契约 | ✅ 已归档 |
| 06 | Prompt 和 Parser 守住两端 | ✅ 已归档 |
| 07 | BaseChatModel 如何统一模型调用 | ✅ 已归档 |
| 08 | init_chat_model 如何动态切换模型 | 未归档 |
| 09（本篇） | **create_agent 如何编译 Agent 运行图** | ✅ **本篇** |

系列共 9 篇，目前已归档 5 篇（02/05/06/07/09）。

## 归档日志

- 2026-07-16 归档
