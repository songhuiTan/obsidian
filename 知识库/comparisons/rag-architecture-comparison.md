---
title: "RAG 架构演进对比 — 从 Naive RAG 到 Agentic RAG"
created: 2026-07-21
updated: 2026-07-21
type: comparison
tags: [comparison, rag, knowledge-base, methodology]
sources:
  - 技能文档/2026-06-20-从原始到Agentic——8种RAG架构深度解析与生产实践指南.md
  - 技能文档/结构树检索RAG_不用向量数据库不做Chunk的RAG方案.md
  - 技能文档/千万文档级RAG逼近零幻觉-10M向量18ms-CRAG四层防线.md
  - 技能文档/2026-06-30-SAG-LLM-Wiki-最强知识库方法论.md
  - 技能文档/Google OKF 企业 AI 上下文架构解析.md
  - 技能文档/2026-06-04-LangGraph-RAG-Memory-MCP企业级AI助手架构.md
confidence: medium
---

# RAG 架构演进对比

## 八大 RAG 架构横向对比

| 架构类型 | 检索方式 | 幻觉控制 | 复杂度 | 适用场景 | 关键能力 |
|----------|----------|----------|--------|----------|----------|
| Naive RAG | 向量相似度检索 Top-K | 弱（依赖 Chunk 质量） | 低 | FAQ、文档问答 MVP | Embedding + LLM |
| Advanced RAG | 多路检索 + 重排序 | 中（RF/重排序过滤） | 中低 | 生产文档问答 | Query 重写、滑动窗口 |
| Modular RAG | 模块化管道（Retrieve/Rerank/Filter） | 中（可组合过滤器） | 中 | 可定制企业问答 | 插件化模块组合 |
| GraphRAG | 向量 + 图遍历（multi-hop） | 高（图结构约束） | 中高 | 多文档/全局合成 | 知识图谱构建 |
| Agentic RAG | Agent 自主决策检索策略 | 高（Planning + 多步验证） | 高 | 复杂推理问答 | Agent Planning/Memory |
| 结构树检索 RAG | 树结构层级检索 | 高（层级约束） | 中高 | 层级明确的知识库 | 无需向量数据库 |
| OKF (Google) | 企业上下文 + 结构化知识 | 高（企业规范约束） | 高 | 企业级 AI 助手 | 上下文架构标准化 |
| SAG + LLM Wiki | Wiki 方法论 + 结构化检索 | 高（Wiki 结构约束） | 中高 | 知识库 Wiki 系统 | 可生长上下文数据库 |

## 幻觉控制与复杂性对比

```
幻觉控制:   低 ----------------> 高
             Naive < Advanced < Modular < GraphRAG < AgenticRAG
复杂度:     低 ----------------> 高

结构树检索 RAG 和 SAG+LLM Wiki 在幻觉控制和复杂度之间取得了较好的平衡，
适合对幻觉敏感但不想引入 Agent 复杂度的场景。
```

## 各架构详细说明

### 1. Naive RAG（朴素 RAG）
最简单的"检索 + 生成"管道。用户查询 -> Embedding -> 向量相似度检索 Top-K -> 拼接上下文 -> LLM 生成。对 Chunk 质量极度敏感，查询-文档语义鸿沟大时效果差。

### 2. Advanced RAG（高级 RAG）
在 Naive 基础上引入查询重写（HyDE、多查询分解）、重排序（Cross-Encoder）、滑动窗口等优化。适合生产环境入门级部署。

### 3. Modular RAG（模块化 RAG）
将检索管道拆解为可组合模块（Retrieve、Rerank、Filter、Fusion 等），支持灵活配置。是很多企业 RAG 平台的基础架构。

### 4. GraphRAG（图检索增强生成）
微软提出，预处理阶段构建知识图谱，检索时兼顾向量相似度和图遍历。在全局、多文档合成任务上表现突出。

### 5. Agentic RAG（智能体 RAG）
引入 AI Agent 的 Planning、Memory、Tool Use、Reasoning。Agent 自主决定何时检索、检索什么、如何综合多源信息。代表最高复杂度但也最强的灵活度。

### 6. 结构树检索 RAG
不使用向量数据库和 Chunk，改用树结构组织知识。适合层级明确的知识领域（如法律、医疗、技术文档）。

### 7. OKF（Google 企业上下文架构）
Google 提出的企业 AI 上下文架构，强调上下文标准化和企业规范约束。

### 8. SAG + LLM Wiki
结合 Wiki 方法论的结构化检索架构，强调知识的可生长性。参考 [[rag-architecture]] 概念文档获取更深度的谱系分析。

## 选型决策树

```
你的场景需要 RAG?
├── 简单文档问答/FAO
│   └── Naive RAG (MVP 足够了)
├── 生产级文档检索
│   ├── 需要低幻觉 -> Advanced RAG + CRAG
│   ├── 多文档全局合成 -> GraphRAG
│   └── 灵活可配置 -> Modular RAG
├── 复杂推理问答
│   └── Agentic RAG (结合 [[langgraph]] 编排)
├── 层级明确的知识库
│   ├── 不需要向量数据库 -> 结构树检索 RAG
│   └── 需要 Wiki 方法论 -> SAG + LLM Wiki
└── 企业级 AI 助手
    └── OKF 架构 + GraphRAG
```

## 实践经验

- 千万文档级别：CRAG 四层防线可实现 10M 向量 18ms 检索
- 私有化部署：需从微服务设计到云原生弹性伸缩全面规划
- 结合 [[deep-agents]] 的子代理编排可与 Agentic RAG 深度结合
- 推荐从 Naive RAG 起步验证，逐步向 Advanced/Modular 演进，最后按需引入 GraphRAG 或 Agentic RAG
