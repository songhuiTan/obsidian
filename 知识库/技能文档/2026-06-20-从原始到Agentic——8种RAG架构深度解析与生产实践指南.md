# 从原始到Agentic：8种RAG架构深度解析与生产实践指南（2026 版）

> **来源：** 公众号「AgenticHub」@南七无名式
> **发布时间：** 2026-06-20
> **原文：** https://mp.weixin.qq.com/s/uhLJKZCLTuMwiAlt0X8YiA
> **标签：** `#RAG` `#LLM` `#AIEngineer` `#AgenticRAG` `#GraphRAG` `#生产落地`

## 概述

RAG 已不再是简单的"检索+生成"管道，而是融合向量、图谱、Agent、多模态的复合智能系统。本文从 Naive RAG 一路拆解到 Agentic RAG，涵盖核心原理、技术细节、适用场景、优缺点、生产落地建议及选型决策框架。

---

## 1. Naive RAG（朴素 RAG）— 一切的起点

**流程：** 用户查询 → Embedding → 向量相似度检索 Top-K → 拼接上下文 → LLM 生成

- 技术要点：chunking（固定长度/递归分割）+ 向量数据库（Pinecone/Weaviate/Milvus）
- 适用：内部文档 FAQ、简单知识库问答、MVP 验证
- 优点：实现成本低、延迟低（<1s）、易调试
- 缺点：对 chunk 质量极度敏感；查询-文档语义鸿沟大时失效
- **生产建议：** 作为 baseline 先跑通评估指标（Faithfulness、Answer Relevance、Context Precision/Recall）

## 2. Multimodal RAG（多模态 RAG）

**原理：** 支持文本、图像、表格、音频、视频等多模态嵌入与检索

- 模型：CLIP、LLaVA、BLIP 等多模态 embedding 模型
- 适用：电商图文问答、医疗影像+报告、PDF 文档智能、多媒体内容理解
- 挑战：多模态 embedding 维度与对齐难度高；存储和检索成本上升
- **落地建议：** 从"图文结合"切入，使用 LLaVA 或 GPT-4o 做统一处理，常结合 OCR + Layout Analysis 预处理

## 3. HyDE（假设文档嵌入）

**原理：** 查询 → LLM 生成"假设的理想答案文档" → 用假设文档 embedding 检索真实 chunk → 丢弃假设文档，只用检索结果

- 适用：查询口语化/简短，知识库文档正式/专业/长文本（如"怎么治失眠"→医学论文）
- 效果：显著桥接语义鸿沟，不匹配查询上 nDCG 等指标提升明显
- 代价：额外一次 LLM 调用增加延迟和成本

## 4. Corrective RAG（CRAG）

**原理：** 检索后增加"验证&纠正"环节，与可信外部来源对比，过滤低可信内容，触发重检索/生成修正

- 适用：金融、法律、医疗、新闻等对事实准确性极高的领域
- 关键组件：可信度评分模型（小型判别器或 LLM-as-Judge）、Web Search API 事实核查、Self-Reflective 机制
- 优势：大幅降低 hallucination，生产环境"可靠性"闭环
- 代价：多轮调用，延迟和 token 成本更高

## 5. Graph RAG（图检索增强生成）

**原理：** 预处理阶段从文档中抽取实体和关系，构建知识图谱。检索时不仅向量相似，还进行图遍历（multi-hop）、社区摘要（community summary）

- 适用：复杂关系推理、企业内部知识网络、科研文献综述、法律条款关联分析
- 微软 GraphRAG 研究：在全局、多文档合成任务上，比传统向量 RAG 有实质性提升
- 挑战：图谱构建成本高；查询时图遍历可能爆炸
- **生产建议：** 混合使用——向量检索召回 + 图检索精排 + 社区摘要压缩上下文

## 6. Hybrid RAG（混合 RAG）

**原理：** 同时使用 Dense Vector（语义）+ Sparse（BM25/关键词）+ Graph/结构化检索，通过 RRF 或加权融合，再 rerank

- 适用：企业搜索（既有非结构化文档又有数据库记录）
- 优势：兼顾"召全率"和"召准率"，目前最务实的生产选择之一
- 实现：LangChain / LlamaIndex 原生支持 Hybrid Retriever

## 7. Adaptive RAG（自适应 RAG）

**原理：** 在检索前/过程中引入路由/分类器，根据查询复杂度动态选择 pipeline

- 简单查询 → 单步 Naive/Advanced
- 复杂查询 → 多步分解 + Graph/Agent
- 技术实现：查询分类器（小型 BERT 或 LLM prompt 分类）、路由机制、不同 pipeline 切换
- 适用：用户查询类型高度多样化的生产系统（客服、内部搜索等）
- 价值：在效果和成本之间实现智能平衡

## 8. Agentic RAG（智能体 RAG）— 最高阶形态

**原理：** 引入 AI Agent 能力——Planning、Memory、Tool Use、Reasoning（ReAct/CoT/ToT）

- Agent 可自主决定：是否检索、检索哪些来源、是否需要多轮迭代、调用外部 API
- 适用：复杂工作流、智能研究助手、自动化报告生成、多数据源整合、多步决策任务
- 关键组件：Planner、Retriever Router、Memory Stream、Critic/Reflector、Executor
- 挑战：延迟高、成本高、可解释性差、可能出现 Agent 循环或漂移
- **生产建议：** 从"轻量 Agentic"（固定工具 + 有限步数）开始，结合 Human-in-the-Loop

## 选型决策框架

| 场景 | 推荐架构 |
|------|----------|
| 起步/MVP | Naive RAG + 强 indexing 优化 |
| 准确性优先 | Corrective RAG + HyDE |
| 关系/推理密集 | Graph RAG 或 Hybrid RAG |
| 查询多样 | Adaptive RAG |
| 复杂工作流/多工具 | Agentic RAG |
| 多模态数据 | Multimodal RAG |
| 企业大规模 | Modular RAG（可插拔组合） |

**底层决定性因素：Indexing 质量。** 如果 indexing 产出的是噪音 chunk，再高级的架构也救不回来。优化 indexing（更好 chunking、元数据、子 chunk + parent 检索、压缩等）是性价比最高的杠杆。

## 评估指标

- Context Precision / Recall
- Answer Faithfulness
- Answer Relevance
- Latency
- Cost per Query
- User Satisfaction

## 战略分析

### 与现有工具链的对照

这篇文章为当前正在积累的 RAG 知识体系提供了**全景式分类框架**。结合已有归档：

| 已有归档主题 | 对应 RAG 架构 |
|-------------|--------------|
| kb-builder / ChromaDB + embedding | Naive RAG（基础层） |
| Trove AI + Obsidian | Naive RAG（个人版） |
| 百亿级私有化RAG系统 | Modular / Hybrid RAG |
| LiteParse PDF 解析 | Indexing 优化层 |
| TencentDB Memory（对话记忆） | Memory Stream（Agentic RAG 组件） |

当前方案（Hermes + ChromaDB + TencentDB Memory）本质上是 **Naive RAG + Agentic Memory** 的混合体，最直接的演进路径是：

1. **短期：** 引入 **HyDE** 提升检索命中率（一句话的桥梁效应）
2. **中期：** 引入 **Adaptive RAG** 路由（简单问题直接查，复杂问题多步拆解）
3. **长期：** 向 **Agentic RAG** 演进（Planner + Tool Use 自洽）

## 相关文档

- [[技能文档/kb-builder — 一键搭建AI知识库（Claude Code Skill + ChromaDB + MCP）]]
- [[技能文档/2026-06-14-干掉NotebookLM-Trove-AI开源项目+Obsidian-本地AI知识库]]
- [[技能文档/2026-06-13-百亿级知识检索的私有化RAG系统-从微服务设计到云原生弹性伸缩]]
- [[教程指南/AI落地6关 — 从技术到业务的完整思考框架]]
