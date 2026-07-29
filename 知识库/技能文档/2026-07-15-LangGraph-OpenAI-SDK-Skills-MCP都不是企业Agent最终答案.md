---
title: "LangGraph、OpenAI SDK、Skills、MCP，为什么它们都不是企业 Agent 的最终答案？"
source: "AI Agent实战派"
source_url: "https://mp.weixin.qq.com/s/m3W1H9Kd0by20CfMfz0Npw"
date: "2026-07-15"
tags: [企业Agent, 架构设计, Intent, Capability Router, 业务编排, 落地实践]
---

> LangGraph、OpenAI SDK、Skills、MCP 分别解决 Workflow 编排、Agent 生命周期、Prompt 管理和 Tool 接入四个不同层次的问题，都不是企业 Agent 的最终答案。真正的答案是一套清晰的**业务编排层 + 五层解耦架构**。

## 核心论点

这些框架不是竞争关系，而是互补关系。企业 Agent 不会只采用其中一种架构，因为每种框架解决的是完全不同层次的问题。

| 框架 | 核心解决的问题 | 企业 Agent 缺失的一层 |
|------|--------------|---------------------|
| LangGraph | Workflow 编排（流程如何执行） | ❌ 不关心业务规则 |
| MCP | Tool 接入（如何连接能力） | ❌ 不关心何时调用哪个 |
| Anthropic Skills | Prompt 管理（能力如何描述） | ❌ 不关心何时加载哪个 Skill |
| OpenAI Agents SDK | Agent 生命周期（如何运行） | ❌ 不关心业务如何决策 |

## 企业 Agent 真正的五层架构

以"退款为什么还没到账"为例，系统真正需要完成五层工作：

```
User Query
     │
     ▼
Intent Recognition（理解用户想做什么）
     │
     ▼
Business State / Business Resolver（查询业务事实）
     │
     ▼
Capability Router（按 Intent + 状态决定用哪些能力）
     │
     ▼
Dynamic Skills（按需加载对应业务知识）
     │
     ▼
Tool Calling（调用真正的系统）
     │
     ▼
LLM Response（组织自然语言回复）
```

### 第一层：Intent
理解用户想做什么（退款、物流、报销、知识查询）。

### 第二层：Business State
查询业务事实（订单、权限、库存、审批状态、员工信息）。**这是最关键也最容易被忽略的一层。** 真正决定系统下一步动作的是业务状态，而不是 Intent 本身。

> 同一个「退款查询」Intent，可能对应完全不同的业务能力：
> - 退款未申请 → Refund Create
> - 退款审核中 → Refund Status
> - 银行处理中 → Payment

### 第三层：Capability Router
根据 Intent + Business State，共同决定需要哪些业务能力。从 Intent→Skill 的直接映射，升级为 Intent + State → Capability → Skill 的显式路由。

### 第四层：Skill
Skill 不再直接对应 Intent，而是对应业务能力（Refund Skill、Payment Skill、Knowledge Skill、Risk Skill）。

### 第五层：Tool
真正的工具调用（CRM、ERP、数据库、搜索、支付接口）。

## 五层解耦的核心价值

每一层都有明确输入输出，独立演进：

- 新增支付方式 → 不修改 Intent
- 新增业务规则 → 不修改 Skill
- 新增 Tool → 不修改 Capability

> LLM 不再负责所有决策。它负责理解语言。业务系统负责业务事实。程序负责能力编排。工具负责执行。

## 与现有工具链的对照

| 维度 | 本文框架 | 当前 Hermes 体系 |
|------|---------|----------------|
| Intent | LLM 识别意图 | WeChat 端输入 → LLM 理解，无显式 Intent 层 |
| Business State | 程序查询业务事实 | 主要通过 Tool 调用和 Memory |
| Capability Router | Intent + 状态 → 能力选择 | Skill 名称/描述由模型自行选择 |
| Skill | 按业务能力组织 | Skill 体系已有，但以功能而非业务域组织 |
| Tool | 统一调用底层系统 | ToolNode 机制，MCP 协议兼容 |

## 战略分析

### 跟之前归档的同一作者文章对比

这篇文章和 07-08 归档的《企业 Agent 如何真正落地——一种面向工业生产的 Agent 架构设计》（同一作者 Jiong / AI Agent实战派）是同一个框架的两次阐述。07-08 篇侧重从开放式 Agent vs 企业 Agent 的差异切入，提出五层架构；本篇则从"现有框架都不够"的角度重新论证同五层架构，更像一个"为什么"的论证补充。

两篇可以互为注释阅读。核心观点一致：**LLM 负责语言，程序负责业务。**

### 对我们的价值

这篇文章的 Capability Router 概念对我们当前体系有直接启示。目前 Hermes 的 Skill 选择主要由模型根据名称/描述自行决定，缺少 Intent + 业务状态的显式路由层。这意味着：
1. 相同 Intent 可能每次都走不同的 Skill，缺乏确定性
2. 模型选错 Skill 的概率随 Skill 数量增长
3. 无法基于业务状态做细粒度的 Skill 编排

如果我们未来要做更专业的企业级场景，增加一个轻量的 Capability Router 层（在模型选 Skill 之前插入 Intent + 业务状态的路由逻辑）是值得考虑的方向。

### 与叶小钗 WorkBuddy 文章的关系

今天归档的三篇文章实际上构成了一个完整的信息链：
1. **叶小钗**拆 WorkBuddy → 生产级 Agent 的完整形态（宏观框架）
2. **小张学AI Agent** 源码解析09 → `create_agent` 具体实现（LangChain 工程落地）
3. **AI Agent实战派** → 五层架构为何超越现有框架（架构哲学）

三篇从不同角度（工程实践、源码实现、架构理论）围绕同一个核心问题：**生产级 Agent 到底应该长什么样？**

## 关联阅读

- [[企业Agent如何真正落地-面向工业生产的Agent架构设计]] — 同一作者 Jiong 07-08 篇，从另一角度阐述五层架构
- [[2026-07-15-拆完WorkBuddy-我看到了生产级Agent的完整形态-叶小钗]] — WorkBuddy 生产级 Agent 框架
- [[2026-07-12-生产级Agent全景-架构Harness工程组织与人才-叶小钗]] — 叶小钗生产级全景

## 归档日志

- 2026-07-16 归档（作者 Jiong / AI Agent实战派）
