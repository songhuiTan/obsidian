---
title: "RAG 架构谱系（RAG Architecture Spectrum）"
created: 2026-07-21
updated: 2026-07-21
type: concept
tags: [rag, knowledge-base, methodology, comparison]
sources:
  - "技能文档/2026-06-20-从原始到Agentic——8种RAG架构深度解析与生产实践指南.md"
  - "技能文档/结构树检索RAG_不用向量数据库不做Chunk的RAG方案.md"
  - "技能文档/千万文档级RAG逼近零幻觉-10M向量18ms-CRAG四层防线.md"
  - "技能文档/2026-06-13-百亿级知识检索的私有化RAG系统-从微服务设计到云原生弹性伸缩.md"
  - "技能文档/2026-06-30-SAG-LLM-Wiki-最强知识库方法论.md"
  - "技能文档/2026-06-04-Blockify-IdeaBlock-RAG前置预处理引擎.md"
  - "技能文档/2026-06-05-OpenViking给Agent建可检索会生长的上下文数据库.md"
  - "技能文档/2026-07-18-DeepAgents深度源码解析与工程实践.md"
  - "技能文档/Google OKF 企业 AI 上下文架构解析.md"
  - "技能文档/2026-06-04-LangGraph-RAG-Memory-MCP企业级AI助手架构.md"
confidence: medium
---

# RAG 架构谱系

RAG（Retrieval-Augmented Generation）已从简单的"检索+生成"管道演进为融合向量、图谱、Agent、多模态的复合智能系统。以下是 RAG 架构从简单到复杂的完整谱系。

## 八大 RAG 架构演进

### 1. Naive RAG（朴素 RAG）
流程：用户查询 → Embedding → 向量相似度检索 Top-K → 拼接上下文 → LLM 生成。适合简单文档 FAQ 和 MVP 验证，但对 chunk 质量极度敏感，查询-文档语义鸿沟大时失效。

### 2. Multimodal RAG（多模态 RAG）
支持文本、图像、表格、音频、视频等多模态嵌入与检索。使用 CLIP、LLaVA、BLIP 等多模态模型。适合电商图文问答、医疗影像+报告等场景。

### 3. HyDE（假设文档嵌入）
查询 → LLM 生成"假设的理想答案文档" → 用假设文档 embedding 检索真实 chunk。显著桥接简短口语化查询与专业文档之间的语义鸿沟。

### 4. Corrective RAG（CRAG）
检索后增加"验证&纠正"环节。通过可信度评分模型过滤低可信内容，触发生成修正或重检索。是降低幻觉的核心架构。

### 5. Graph RAG（图检索增强生成）
预处理阶段构建知识图谱，检索时兼顾向量相似度和图遍历（multi-hop）。微软 GraphRAG 在全局、多文档合成任务上表现突出。

### 6. Hybrid RAG（混合 RAG）
同时使用 Dense Vector（语义） + Sparse（BM25/关键词） + 图/结构化检索，通过 RRF 或加权融合，兼顾"召全率"和"召准率"。

### 7. Adaptive RAG（自适应 RAG）
引入查询分类器，根据查询复杂度动态选择 pipeline：简单查询走单步 Naive，复杂查询走多步分解 + Graph/Agent。

### 8. Agentic RAG（智能体 RAG）
最高阶形态，引入 AI Agent 的 Planning、Memory、Tool Use、Reasoning 能力。Agent 自主决定何时检索、检索什么、如何综合多源信息。

## 前沿变体

**结构树检索 RAG**：不依赖向量数据库和 Chunk，改用树结构组织知识，适合层级明确的领域。

**SAG / OKF 架构**：强调将 Wiki 方法论引入 RAG 系统，构建可生长的上下文数据库。

**OpenViking**：为 Agent 构建可检索、会生长的上下文数据库。

## 生产实践要点

- 千万文档级别的 CRAG 可实现 10M 向量 18ms 检索
- 私有化 RAG 系统需从微服务设计到云原生弹性伸缩全面规划
- [[langgraph]] 结合 RAG 与 MCP 构建企业级 AI 助手
- [[deep-agents]] 的子代理编排可与 Agentic RAG 深度结合
