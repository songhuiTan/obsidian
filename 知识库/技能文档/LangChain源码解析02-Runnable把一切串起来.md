---
source: 微信公众号
title: "LangChain源码解析02：Runnable把一切串起来"
author: 小张学AI Agent
date: 2026-07-08
url: https://mp.weixin.qq.com/s/cOYJN_7pZ3FZbVRdAD95ww
tags:
  - LangChain
  - Runnable
  - LCEL
  - 源码分析
  - Agent工程化
  - 流式
---

# LangChain源码解析02：Runnable把一切串起来

> LangChain 不是先有一堆 chain，再把它们包装成 Runnable；它是先把"一段可执行工作"抽象成 Runnable，再让 prompt、model、parser、tool、retriever 都能被放进同一条工程链路。

## 一、Runnable 先统一"怎么运行"

Runnable 的第一层价值不是组合，而是统一运行入口。一个对象只要是 Runnable，就必须回答：

| 能力 | 方法 | 说明 |
|------|------|------|
| 单个输入→输出 | `invoke` / `ainvoke` | 基本调用 |
| 批量处理 | `batch` / `abatch` | 多个输入一起处理 |
| 流式输出 | `stream` / `astream` | 边生成边输出 |
| 流式转换 | `transform` / `atransform` | 上游流→下游流 |
| 运行时上下文 | tags / metadata / callbacks / configurable | 配置传递 |

> **默认实现 ≠ 高性能实现：** 默认 `ainvoke` 把同步 `invoke` 放进 executor；默认 `batch` 并发调用多个 `invoke`；默认 `stream` 只是 `yield self.invoke(...)`。真正的流式能力，要看它有没有覆盖 `stream`，更要看组合链路里有没有实现 `transform`。

## 二、`|` 不是拼接，是生成 RunnableSequence

```python
chain = prompt | model | parser
result = chain.invoke({"topic": "LangChain"})
```

`__or__` / `pipe()` 最终创建 `RunnableSequence`。它做的事不止输入→输出传递：

- **根 run → 子 callback**：每经过一个步骤，用 `seq:step:n` 创建子 callback，链路是**可追踪的运行树**
- **批处理按步骤推进**：第一层处理所有输入 → 第二层再处理第一层所有输出，保留组件的批处理优化
- **保留结构信息**：批处理、异步、流式和 tracing 全在结构中

## 三、字典 = 并发分支

```python
chain = retriever | {
    "answer_context": format_docs,
    "raw_documents": lambda docs: docs,
}
```

dict 被规整为 `RunnableParallel`。运行时行为：
- 复制当前 steps
- 用 executor **并发提交每个分支**
- 每个分支创建 `map:key:<key>` 子 callback

**你写的是 dict，运行时拿到的是并发可追踪的结构。**

## 四、coerce_to_runnable — LCEL 的入口闸门

| 输入类型 | 包装结果 |
|----------|----------|
| 已是 Runnable | 原样返回 |
| 普通 callable | `RunnableLambda` |
| generator function | `RunnableGenerator` |
| dict | `RunnableParallel` |
| 其他 | 类型错误 |

> 方便不代表没有代价。`RunnableLambda` 适合非流式小变换；如果逻辑要保留 chunk 增量输出，用 generator 或自定义 `transform`。

## 五、真正的流式，卡在 transform

**实用判断：**
- 逻辑是"拿完整文本再算一次" → `RunnableLambda`
- 逻辑是"每来一段就处理一段" → `RunnableGenerator` 或自定义 `transform`

`RunnableLambda` 适合顺手接入 callable，但默认阻塞流式，要等完整输入后才产出结果。
`RunnableGenerator` 接受 `Iterator[A] -> Iterator[B]`，上游 chunk 到来时就处理并吐出，适合自定义 parser、token 清洗、增量格式化。

**多个阻塞组件存在时，流式在最后一个阻塞组件之后才开始。**

## 六、RunnableConfig — 运行时上下文

每个核心方法都接受 `config`，不是参数袋，而是整条链路的运行时上下文：

- `ensure_config` 补齐默认字段：tags、metadata、callbacks、recursion_limit、configurable
- 非标准 key 自动放进 `configurable`，实现运行时切换模型、传递业务参数
- `patch_config` 进入子步骤时改写上下文（替换 callbacks、调整 recursion limit、设置 run name）

没有它，`prompt | model | parser` 只能得到一个结果；有了它，才能知道每一步怎么运行、哪里耗时、哪里报错。

## 七、读 Runnable 源码的四个问题

1. 输入输出是什么？有没有 schema？
2. 是原生 Runnable，还是被 `coerce_to_runnable` 包装出来的？
3. 在 sequence 或 parallel 里生成什么运行结构？
4. 有没有真正实现 `transform`，还是会在流式链路里形成缓冲点？

## 八、总结

> Runnable 是 LangChain 标准对象共同遵守的执行协议。它把一段 LLM 应用逻辑变成可调用、可批处理、可流式、可组合、可追踪、可配置的工程对象。LCEL 的简洁，来自这层协议的复杂；上层体验越轻，底层约束就越重。
