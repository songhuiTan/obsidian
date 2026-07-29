---
source: 微信公众号
author: 因吹斯听-路路
account: 因吹斯听-路路
date: 2026-06-24 09:02
url: https://mp.weixin.qq.com/s/v1xZjJ6U5Z6_psDrHUgLSA
tags: LangGraph, DeepAgents, Hermes Agent, Agent框架, 多Agent编排, 源码分析, 框架对比
---

# LangGraph vs DeepAgents vs Hermes：三大Agent框架核心源码拆解分析

> 用别人的轮子做自己的应用。三种完全不同的技术路线，搞清楚它们的核心实现，对多Agent系统设计的理解会上升一个层次。

---

## 一、LangGraph：把Agent编排变成画流程图

核心思想：**用有向图描述Agent的执行流程**。每个节点是一个计算单元，每条边是一次状态转移，条件边是动态路由。

### 1.1 图状态机的核心抽象

三个抽象：`StateGraph`、`Node`、`Edge`。

```python
from typing_extensions import TypedDict
from langgraph.graph import StateGraph, START, END

class AgentState(TypedDict):
    messages: list
    current_task: str
    results: dict

workflow = StateGraph(AgentState)

def planner_node(state: AgentState):
    plan = llm.invoke(state["messages"])
    return {"current_task": plan}  # 只返回要更新的字段
```

**设计哲学：节点之间完全解耦**——每个节点只关心自己要更新的字段，不需要知道其他节点的存在，与微服务设计哲学如出一辙。

### 1.2 条件边：动态路由的精髓

```python
def route_by_quality(state: AgentState):
    score = evaluate(state["results"])
    if score > 0.8:
        return "finalize"    # 质量达标，直接输出
    else:
        return "improve"     # 质量不够，继续优化

workflow.add_conditional_edges(
    "evaluator", route_by_quality,
    {"finalize": END, "improve": "optimizer"}
)
```

可实现：**重试循环、质量门控、多路径分支**，流程可视化（LangSmith Studio）。

### 1.3 多Agent协作：Send API与子图

**Send API**——运行时动态创建Worker实例，并行派发 + 归并器聚合：

```python
def assign_workers(state: AgentState):
    return [Send("worker_node", {"section": s}) for s in state["planned_sections"]]

class State(TypedDict):
    completed_sections: Annotated[list, operator.add]  # 归并策略
```

**子图（Subgraph）**——把一个完整的StateGraph作为另一个图的节点，实现层级化编排。

### 1.4 持久化与人机协作

- **Checkpointer**：线程级图状态快照，支持断点恢复、时间旅行、Human-in-the-Loop（`interrupt()`）
- **Store**：跨线程长期记忆，命名空间隔离

**核心范式：显式控制**——精确定义每一步状态转移，适合需要严格流程控制、审计追踪、人机协作的场景。

---

## 二、DeepAgents SDK：从浅层工具调用到深度自主

TypeScript框架，基于Vercel AI SDK v6，无LangChain依赖。核心论点：大多数Agent只是简单的工具调用循环，遇到复杂任务就废了。

### 2.1 深度Agent的四大支柱

| 支柱 | 说明 |
|------|------|
| **Planning** | 内置 `write_todos`，Agent必须先拆解任务、生成待办清单 |
| **Subagents** | 内置 `task` 工具派发子任务，独立上下文/工具集/模型 |
| **Virtual Filesystem** | 中间结果写入文件而非上下文，解决上下文膨胀 |
| **Detailed Prompts** | 每个子Agent独立system prompt |

### 2.2 子Agent声明式配置

```typescript
const agent = createDeepAgent({
  subagents: [
    {
      name: "researcher",
      description: "负责搜索和整理资料",
      systemPrompt: "你是一个专业的研究员...",
      tools: [webSearch, readFile],
      model: "gpt-4o",
      interruptOn: { webSearch: true },   // 搜索前需人工确认
      output: researchOutputSchema,        // Zod schema约束输出
    },
    {
      name: "writer",
      description: "根据资料撰写文章",
      systemPrompt: "你是一个技术写作专家...",
      tools: [writeFile],
      model: "claude-sonnet",
    }
  ]
});
```

**精妙之处：**
- `interruptOn`：工具级人工审批，粒度细到单个工具调用
- `output schema`：Zod约束格式，父Agent拿到结构化数据
- **模型隔离**：不同子Agent用不同模型，成本和能力优化

### 2.3 杀手锏：虚拟文件系统

Agent把中间结果写入文件而非塞在上下文里。典型执行轨迹：

```
/workspace/plan.md       → 任务规划
/workspace/research.md   → 研究笔记
/workspace/draft.md      → 初稿
/workspace/final.md      → 终稿
```

子Agent返回精简摘要，详细结果存文件——和人类的工作方式一致。

### 2.4 流式事件与可观测性

```typescript
for await (const event of agent.streamWithEvents({ prompt })) {
  switch (event.type) {
    case "subagent-start":     // 子Agent启动
    case "tool-call":          // 工具调用
    case "file-written":       // 文件写入
    case "subagent-complete":  // 子Agent完成
  }
}
```

可构建实时进度UI：用户看到「研究员正在搜索… → 资料已保存 → 写手开始撰写…」。

**核心范式：深度自主 + 状态外置**——通过规划前置、子Agent声明式配置、虚拟文件系统解决上下文膨胀，适合长周期复杂任务。

---

## 三、Hermes Agent（简述）

本文未展开，详见后续文章《Hermes Agent v0.18.0 深度解析》。

**核心范式：环境构建**——不是编排Agent，而是给Agent一个能学习、能记忆、能社交、能自我改进的运行环境。

---

## 写在最后：三条路线

| 框架 | 路线 | 适合场景 |
|------|------|---------|
| **LangGraph** | 显式控制 | 需要确定性和可审计性的生产环境 |
| **DeepAgents** | 深度自主 | 长周期复杂任务，状态外置解决上下文膨胀 |
| **Hermes Agent** | 环境构建 | 让Agent学习/记忆/社交/自我改进 |

> 技术选型的核心不是「哪个框架更强」，而是「我的问题的本质是什么」。

**资源链接：**
- LangGraph: docs.langchain.com
- DeepAgents SDK: github.com/nicholasgriffintn/deepagents-sdk
- Hermes Agent: hermes-agent.nousresearch.com/docs
