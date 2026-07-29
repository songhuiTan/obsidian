---
source: 微信公众号
author: AI砖家成长日记
account: AI砖家成长日记
date: 2026-06-12 08:35
url: https://mp.weixin.qq.com/s/2gM4g4_6OGe30KLDPuf4LQ
tags: DeepAgents, LangGraph, Agent框架, create_deep_agent, 实践对比
---

# 用 LangGraph 写 Agent 写到第三个，我决定看看 DeepAgents 到底替我做了什么

> Day 1 笔记：搞清楚 DeepAgents 到底是什么、不是什么，以及如果你已经会 LangGraph，到底要不要花时间学它。

---

## 一、DeepAgents 和 LangGraph 的关系

**不是替代品，是同一栈的不同抽象层。**

| 层 | 说明 |
|----|------|
| **LangGraph** | 底层编排引擎。让你完全控制图结构、状态转移、条件边、中断恢复 |
| **create_agent** | 在其上封装了标准 ReAct 循环。省去手写 while 循环 + tool calling |
| **create_deep_agent** | 再往上封装了 Sub-agents、Filesystem、Memory、Shell、Skills、HITL 等工业级能力 |

三者可叠加使用。**任何 LangGraph 的 CompileGraph 都可以当 Sub-agent 塞进 Deep Agent**——过去用 LangGraph 写的复杂图可以直接挂上来当子能力。

---

## 二、DeepAgents 解决了什么

| 能力 | LangGraph 里要自己做的事 | DeepAgents 默认带 |
|------|------------------------|------------------|
| Sub-agents | 自己 compile 多个 graph，定义之间怎么调 | 直接配置，子 Agent 上下文隔离 |
| Filesystem | 自己写 read/write tool，处理后端 | 内置文件读写工具，可换本地/沙箱/远程 |
| Context Management | 自己写摘要节点、决定何时压缩 | 长对话自动摘要、工具结果自动落盘 |
| Persistent Memory | 自己接 checkpointer/store | State 和 Store 均可插拔 |
| Shell | 自己包装 subprocess 工具 | 内置沙箱 shell |
| Skills | 没有这个抽象 | 按需加载的可复用行为包 |
| HITL | 自己用 interrupt 机制 | 工具调用前可 approve/edit/reject |
| Tools | LangChain Tool 或自定义 | 兼容 LangChain Tool + MCP Server |

**核心定位：** 把「长任务、多步骤 Agent 项目里你每次都要重写一遍」的那部分，做成了默认能力。

---

## 三、30 秒跑出第一个 Deep Agent

### 3.1 安装
```bash
uv add deepagents
# 或 pip install deepagents
```

### 3.2 最小 Agent
```python
from deepagents import create_deep_agent

def get_weather(city: str) -> str:
    """查询某个城市的天气（演示用，写死）。"""
    return f"{city} 今天 26°C，多云。"

agent = create_deep_agent(
    model="openai:gpt-4o",          # 任何支持 tool calling 的模型
    tools=[get_weather],            # 也支持 MCP
    system_prompt="你是一个研究助手，回答问题前先想清楚是否需要工具。",
)

result = agent.invoke({"messages": "北京天气怎么样？"})
print(result["messages"][-1].content)
```

### 3.3 观察什么
1. `result["messages"]` 里 Agent 自己产生了哪些消息（有没有反思/计划）
2. 长任务是否主动调用文件系统写中间产物
3. LangSmith trace 中底层确实就是 LangGraph

**和手写 LangGraph ReAct 的差异：** 文件读写工具不用你写，上下文超长自动摘要不用你写，子任务委派不用你写图，工具拦截审批配置就有。

---

## 四、什么时候不该用

**不适合：**
- Agent 循环不是 ReAct/Plan-Execute 形状（严格 DAG/状态机/事件驱动）→ 直接 LangGraph
- 追求极致最小依赖 → `create_agent` 更轻
- 强约束场景，不能让模型自己决定文件读写 → DeepAgents 核心假设是"trust the LLM"

**适合：**
- 长任务、多步骤，希望 Agent 自己规划自己执行
- 想做"Claude Code 那种通用工作型 Agent"（灵感来源）
- 已经有一堆 LangGraph 子图，想被一个上层 Agent 调度

---

## 五、踩坑预告

1. **模型必须支持 tool calling** — 本地 ollama 小模型工具调用经常空转
2. **额外 SDK 依赖** — `pip install deepagents` 之外还要 `pip install langchain-openai` 之类
3. **默认带的工具不少** — 第一次跑别急着加自己的 tool，先看默认装了啥

---

## 六、Day 1 一句话结论

> **DeepAgents 不是新 Agent 框架，它是 LangGraph + create_agent 之上的「工业级默认配置」。**
> 
> 如果你已经会 LangGraph，学 DeepAgents 的成本是几小时，省下来的样板代码是每个项目几百行。

**值不值得学：** 你最近的 Agent 项目里，是不是已经第二次在写文件读写工具、上下文摘要、子任务委派？是就学，不是就留着下次再看。
