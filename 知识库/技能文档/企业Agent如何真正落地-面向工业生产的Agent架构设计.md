---
source: 微信公众号
title: "企业 Agent 如何真正落地？一种面向工业生产的 Agent 架构设计"
author: Jiong（AI Agent实战派）
date: 2026-07-08
url: https://mp.weixin.qq.com/s/jcmbBFxpOQ0c_RrSet-oVg
tags:
  - 企业Agent
  - 架构设计
  - Intent
  - Intent Recognition
  - Capability Router
  - Dynamic Skill
  - 业务编排
  - 落地实践
---

# 企业 Agent 如何真正落地？一种面向工业生产的 Agent 架构设计

> Demo 很容易，真正上线却很难。问题并不是模型能力不足，而是企业 Agent 与开放式 Agent 面临的是两类完全不同的问题。
> 
> **LLM 负责语言，程序负责业务。**

## 开放式 Agent vs 企业 Agent

| | 开放式 Agent | 企业 Agent |
|---|---|---|
| 任务类型 | 未知，探索式（Exploration） | 确定性业务流程 |
| 核心 | Planner（规划下一步） | 业务能力组织 |
| 示例 | "帮我定位仓库编译失败" | "我的订单什么时候发货？" |
| LLM 角色 | 不断推理和规划 | 理解语言 + 生成回复 |
| 真正决定答案的 | LLM 的推理过程 | 业务系统的数据 |

## 五层企业 Agent 架构

```
User Query
     │
     ▼
Intent + Slot（LLM）
     │
     ▼
Business Resolver（程序）
     │
     ▼
Capability Router（程序）
     │
     ▼
Dynamic Skill Builder（程序）
     │
     ▼
Base Prompt + Skills + Context
     │
     ▼
LLM Final Response
```

### 第一层：Intent Recognition（LLM）

**职责仅有两个：**
1. 理解用户想做什么（Intent）
2. 提取业务参数（Slot）

```json
{
  "intent": "refund_query",
  "slots": {
    "order_id": null
  }
}
```

> Intent 不负责判断退款是否成功、订单是否存在、调用哪个接口——这些属于业务逻辑。

### 第二层：Business Resolver（程序）

查询所有**确定性的业务事实**：
- 数据库 / CRM / ERP / OA / HR
- 权限系统 / Redis / 配置中心

**这一层完全不需要 LLM，全部由程序完成。**

### 第三层：Capability Router（程序）

**关键洞察：真正决定需要哪些能力的，不是 Intent，而是业务状态。**

同是 `refund_query`，不同状态下需要的能力完全不同：

| 业务状态    | 需要的能力               |
| ------- | ------------------- |
| 退款申请未提交 | Refund Create Skill |
| 退款审核中   | Refund Status Skill |
| 银行处理中   | Payment Skill       |

Capability Router 根据 **Intent + Business State** 共同决定需要哪些业务能力。

### 第四层：Dynamic Skill Builder（程序）

Skill = 业务能力模块，不是 Prompt。

- 售后：Refund Skill
- 物流：Logistics Skill
- 商品：Product Skill
- 知识库：Knowledge Skill

系统动态拼接 Base Prompt + 相关 Skill，而不是把所有业务规则放到一个巨大的 System Prompt 中。**显著降低 Prompt 长度，提高 Prompt Cache 命中率，降低维护成本。**

### 第五层：Runtime Context Builder（程序）

Context 也动态生成。**Capability 决定 Context。**

- 退款能力 → 只需要：订单信息 + 退款状态
- 物流能力 → 只需要：物流单号 + 物流轨迹
- 知识库 → 只需要：召回文档

## 为什么五层架构更好做 SFT？

整个流程天然形成监督数据：

| 层                 | 产生的数据                                          |
| ----------------- | ---------------------------------------------- |
| Intent            | `{"intent":"refund_query"}`                    |
| Business Resolver | `{"refund_status":"bank_processing"}`          |
| Capability        | `{"capabilities":["refund_status","payment"]}` |
| Final Response    | "您的退款已经完成审核，银行正在处理中…"                          |

Intent Model / Decision Model / Capability Router / Response Model 都可以分别进行 SFT，甚至通过强化学习优化。**整个 Agent 不再是一个巨大的黑盒，而是一系列可观测、可训练、可评估的模块。**

## 核心原则

| 擅长者 | 负责内容 |
|--------|---------|
| **LLM 擅长** | 理解自然语言、理解上下文、组织表达、处理开放问题 |
| **程序擅长** | 查询业务数据、判断业务状态、权限控制、Tool 路由、Skill 加载、工作流编排 |

> 只有把**语言智能**和**业务智能**彻底解耦，企业 Agent 才能真正从 Demo 走向生产。
