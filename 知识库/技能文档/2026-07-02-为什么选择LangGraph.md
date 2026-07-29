---
title: "为什么选择 LangGraph？"
source: "大白话讲AI"
source_url: "https://mp.weixin.qq.com/s/vKviHyRIwILbmUaNKlICPg"
date: "2026-07-02"
tags: [LangGraph, StateGraph, 企业Agent, 教程]
---

> Demo 能跑只是开始，生产可控才是企业级 AI 应用的分水岭。LangGraph 的核心不是让模型更聪明，而是把 AI 能力放进可控、可恢复、可维护的业务流程里。

## LangGraph 心智模型

**`State`** — 不是临时变量，而是整张图的共享工作台

**`Node`** — 不是人类脑中的大步骤，而是接收 State、返回更新的执行单元。节点越薄，恢复时越不容易重复调用外部 API，也越容易在中间插入安全边界

**`Edge`** — 不是随口判断，而是显式控制流（普通边 + 条件边）

**`Reducer`** — 不是语法装饰，而是旧状态和新状态的合并规则（如 `add_messages` 追加去重）

## 旅行计划助手示例

```
parse_request → extract_trip_info → build_plan → save_plan → reply_plan
```

关键：`travel_plans: Annotated[list[dict], add]` — 新计划追加而非覆盖。

## 为什么不是 CrewAI/AutoGen/Agno？

| 框架 | 适合场景 |
|------|---------|
| CrewAI | 角色扮演式协作（研究员+分析师+写作者） |
| AutoGen | 多 Agent 对话实验 |
| Agno | 快速 Demo |
| LangGraph | **编排层** — 传统业务代码和 AI 节点可放在同一张图里 |

> LangGraph 的核心不是让 Agent 更像人，而是让 AI 和传统业务代码可以在同一个流程里被清楚地编排。

## 什么时候不该用

简单问答机器人、五分钟 Demo、纯角色协作、Java 技术栈（Spring AI 更适合）时 LangGraph 可能太重。

## 归档日志

- 2026-07-16 归档
