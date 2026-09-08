# AI兵器谱第47期：ADK Python（Google Agent 开发套件）

> 来源：AI架构前哨｜抖音 2026-08-01
> 原文：https://www.douyin.com/video/7668851750450908450
> GitHub：[google/adk-python](https://github.com/google/adk-python)（20.9K⭐ / 3.8K forks / Python / v2.6.1）
> 标签：AI, ADK, Agent, Google, 框架, 工具包, 视频归档

---

## 概述

AI兵器谱第47期介绍 **Google ADK Python**（Agent Development Kit），江湖人称"星罗阵"。截至 2026-08-01：20.9k stars、3.8k forks、Python、最新 release v2.6.1。

定位：Google 开源的 **code-first Agent 工具包**，2.0 版本引入 graph-based Workflow Runtime 和结构化 Task API。

## 核心内容

### ADK 2.0 新特性

- **Workflow Runtime**：基于图的执行引擎，用于组合 agentic 应用的确定性执行流
- 支持：路由、fan-out/fan-in、循环、重试、状态管理、动态节点、HITL（人在回路）、嵌套工作流
- **结构化 Task API**：任务协作模型

### 设计哲学

| 维度 | ADK v1 | ADK 2.0 |
|------|--------|---------|
| 模型实例化 | 直观的 model instantiation | 保留 |
| 编排 | 回调控制 + context 抽象 | **graph-based Workflow Runtime** |
| 协作 | — | 结构化 Task API |
| 语言 | Python / Java / Go / TS / Kotlin | Python 先行，Go 2.0 已发布 |

### 光谱式控制（Spectrum of Control）

ADK 2.0 的核心主张：**无缝融合 Agent 的探索能力与确定性执行逻辑的可靠性**——在"自由探索的 Agent"和"严格确定的 Workflow"之间提供连续光谱，而非二选一。

## 快速上手

```python
from google.adk import Agent
from google.adk.tools import google_search

agent = Agent(
    name="researcher",
    model="gemini-flash-latest",
    instruction="You help users research topics thoroughly.",
    tools=[google_search],
)
```

安装：`pip install google-adk`

## 战略分析

### 与现有体系的对照

| 维度 | Google ADK | Hermes Agent | 说明 |
|------|-----------|-------------|------|
| 定位 | code-first Agent 开发套件 | 通用 Agent 框架 | ADK 更偏开发 SDK |
| 编排 | graph-based Workflow Runtime | 工具循环 + Skill | ADK 2.0 的图执行引擎更强 |
| 任务协作 | 结构化 Task API | delegate_task 子代理 | 概念对应 |
| HITL | 内置 human-in-the-loop | 部分场景 | ADK 原生支持 |
| 模型 | Gemini 优先，多模型 | 多 provider | ADK 绑 Google 生态 |
| 部署 | Agent Runtime (Google Cloud) | 本地/自托管 | ADK 有云原生部署路径 |

### 值得借鉴的点

1. **Graph Workflow Runtime 是 ADK 2.0 的核心竞争力**——把"确定性执行流"（路由/并行/循环/重试/HITL）做成图的运行时。这与近期归档的 Harness 引擎、LangGraph 等思路同源，但 ADK 把它做成了产品级的完整实现

2. **Spectrum of Control 主张**：Agent 和 Workflow 不是二选一，而是一个连续谱。复杂任务需要"确定性流程 + 自由探索"的混合——这个设计哲学对当前 Hermes 的 Skill 体系有直接参考价值

3. **"江湖人称星罗阵"**：比喻 ADK 像星罗棋布的法阵，把 Agent 布成网络协同工作——呼应多 Agent 编排的趋势

### 能力差距

- ADK 2.0 的 Workflow Runtime 是当前 Hermes 缺少的（Hermes 有 Skill 但无图执行引擎）
- ADK 绑定 Gemini/Google Cloud，Hermes 的优势是 provider 中立 + 本地部署
- 视频系列"AI兵器谱"持续更新，后续期数值得关注

## 归档日志

- 2026-08-01 归档
（抖音视频页需登录，通过浏览器提取元数据 + 官方文档补充技术细节）
