---
source: 微信公众号
title: 讲透 LangGraph：从状态图到 Agent 工程化 01｜为什么需要 LangGraph：Agent 复杂之后，链式调用不够用了
author: 张龙埋怨 / 小张学AI Agent
date: 2026-07-07
category: AI Agent 框架与工程化
tags: LangGraph, StateGraph, Agent, Pregel, 状态图
link: https://mp.weixin.qq.com/s/2syNP12IbRJrsS5IeiJKrA
status: 已归档
---

# 讲透 LangGraph：从状态图到 Agent 工程化 01｜为什么需要 LangGraph

> 作者：张龙埋怨 / 小张学AI Agent | 2026-07-07
> 公众号：小张学AI Agent
> 原文：[微信链接](https://mp.weixin.qq.com/s/2syNP12IbRJrsS5IeiJKrA)

---

## 核心观点

LangGraph 不是为了把简单流程写复杂，而是为了把本来就复杂的 Agent 流程显式表达出来。复杂 Agent 的核心不是模型调用，而是状态流转。

---

## 一、链式调用的三个问题

### 1. 状态不好管理
真实 Agent 需要知道：当前步骤、已调用工具、用户补充信息、模型推理结果、失败记录、保留给下一轮的状态。全部塞进函数参数 → 代码变成"参数传来传去"的毛线球。

### 2. 流程不是直线
Agent 逻辑：模型需要工具 → 调用工具 → 结果不够 → 继续判断 → 得到答案 → 结束 / 高风险 → 人工确认 / 失败 → 重试或兜底。这是**图**，不是**链**。

### 3. 执行过程需要保存
生产环境 Agent 应做到：中断后继续、失败后恢复、人类介入后从原位置接着跑、回到历史状态重新分叉执行。"多写几个 if else" 解决不了。

---

## 二、LangGraph 的核心判断

> **节点通过读取和写入共享状态来通信。**

| 维度 | 链式调用 | 状态图 |
|------|---------|--------|
| 关心 | A 的输出给 B，B 的输出给 C | 当前状态是什么，哪些节点被触发，执行后状态如何变化 |
| 数据流 | 函数参数传递 | 共享 State（白板模型） |
| 控制流 | 线性/固定 | 图（分支+循环+暂停+恢复） |

---

## 三、三个核心概念

### 1. State — 整张图共享的状态
聊天 Agent 示例：`messages`, `remaining_steps`, `user_id`, `tool_results`, `final_answer`
状态字段可定义更新规则（追加 vs 覆盖 vs 累加）→ Reducer 和 Channel

### 2. Node — 执行单元
`State -> Partial State`：读入当前状态，返回一部分状态更新。不是"下一个节点的入参"，而是对 State 的更新。

```python
def call_model(state):
    messages = state["messages"]
    response = model.invoke(messages)
    return {"messages": [response]}
```

### 3. Edge — 节点间的流动
- 固定边：`START -> call_model -> END`
- 条件边：模型需要工具 → tools / 否则 → END
- 循环：`agent -> tools -> agent -> tools -> agent -> END`

---

## 四、Pregel — 真正的执行引擎

`StateGraph` 是 builder，`Pregel` 是核心执行引擎，借鉴 **Bulk Synchronous Parallel**（BSP）模型：

每一轮（Superstep）：
1. **Plan** — 决定本轮哪些节点要执行
2. **Execution** — 并行执行这些节点
3. **Update** — 把节点产生的更新写回 Channel 和 State

本质变化：从"上一个函数调用下一个函数" → **"根据上一轮状态变化，决定下一轮哪些节点应该被触发"**

---

## 五、现实项目中的 Agent 工程化问题

- 用户任务不是一步完成的
- 模型决策路径不稳定
- 工具调用可能失败
- 状态需要跨轮保存
- 中途要接入人工审核
- 流式输出要实时反馈
- 历史执行过程要能追踪
- 出问题后要能恢复和排查

LangGraph 把这些变成**框架级能力**，而非在业务代码里打补丁。

---

## 六、系列规划（共 20 篇）

| 阶段 | 内容 |
|------|------|
| 第一阶段 | 核心概念：项目结构、StateGraph 建图、State/Node/Edge、Reducer 和 Channel |
| 第二阶段 | 执行引擎：compile()、Pregel、invoke() 完整链路、stream |
| 第三阶段 | 工程能力：interrupt、checkpoint、time travel、subgraph、runtime |
| 第四阶段 | 上层封装：create_react_agent、ToolNode、CLI & SDK、框架设计启发 |

---

## 总结

> 当任务还是一条直线时，Chain 很好用。
> 当任务开始有状态、有分支、有循环、有中断、有恢复、有人工介入时，Graph 更接近问题本身。
> **复杂 Agent 的核心，不是模型调用，而是状态流转。**
