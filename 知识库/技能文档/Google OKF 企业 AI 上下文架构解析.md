---
source: "AI研究生 / AI大模型观察站"
url: "https://mp.weixin.qq.com/s/GwuOid_f1VZeocn8SWWLzg"
date: 2026-07-13
tags: OKF, Knowledge Graph, RAG, Google, AI上下文, 企业架构
---

# 从 Vector Retrieval 到 Knowledge Graph：Google OKF 的企业 AI 上下文架构解析

> Google 开源 Open Knowledge Format (OKF v0.1)，以 Markdown、YAML 与知识图谱替代纯向量检索，为企业 AI agent 提供更确定、可审计的上下文管理方式。

## 背景：RAG-everything 的裂痕

过去三年，企业 AI 上下文的默认方案是「构建 RAG pipeline」：启动向量数据库、对 PDF 进行 chunk、生成 embeddings、运行时执行 semantic similarity search。但到 2026 年，RAG-everything 的问题已无法忽视：

- Chunking 破坏复杂的表格结构
- Vector retrieval 本质上是概率性的（可能拿到正确 chunk，也可能拿到过时的）
- Embeddings 与快速更新的数据保持同步是运维噩梦

## Open Knowledge Format (OKF) 是什么

Google Cloud 开源的 **OKF v0.1** 是一个 vendor-neutral、可移植的规范，用于形式化 **"LLM Wiki" 范式**——结构化、互联的"大脑"概念。

OKF 将策略从**在非结构化文件上进行概率式搜索**，转变为**在一个鲜活的、同时可供人类和 agent 阅读的 knowledge graph 中进行确定性导航**。

### OKF Bundle 的结构

一个 Knowledge Bundle 只是一个由纯文本 Markdown 文件组成的标准目录，用 YAML frontmatter 包裹。目录路径定义概念的唯一身份。

```
company_brain/
├── index.md               # Root directory for progressive disclosure
├── engineering/
│   ├── index.md
│   └── service_mesh.md    # Architecture concept
└── analytics/
    ├── index.md
    ├── tables/
    │   ├── customers.md   # Individual database concept file
    │   └── billing.md
    └── metrics/
        └── active_users.md # Precise business definition
```

每个 concept 文件：顶部是 **YAML frontmatter**（只要求 `type` 字段），随后是自由格式的 **Markdown body**。

```yaml
---
type: metric
id: analytics/metrics/active_users
title: Weekly Active Users (WAU)
owner: data-eng@company.com
updated_at: 2026-06-15
citations:
  - source: "https://github.com/internal-org/dbt/models/wau.sql"
---
```

## OKF 的三大核心支柱

1. **Format over Platform**：git-native，可版本控制、通过 PR 审计、精确追踪知识变化
2. **LLM as the Wiki Librarian**：后台 AI agents 自动维护文档，修复 cross-links，更新 log.md
3. **通过 Graph Links 实现严格确定性**：使用显式 Markdown links（`[[concept_path]]`）替代 cosine similarity，将文件夹转化为绝对的 Knowledge Graph

## 正面对比：RAG vs. OKF

| 维度 | RAG | OKF |
|------|-----|-----|
| 核心结构 | 分段、碎片化的 vector chunks | 结构化 Markdown + YAML frontmatter |
| 检索引擎 | 概率式（数学 nearest-neighbor） | 确定性（显式 graph link traversal） |
| 人机可读性 | 低（需要查询工程 DB） | 高（可在 GitHub 或 Obsidian 中原生阅读） |
| 维护成本 | 高（重新索引、embedding drift） | 低（Git commits 和 PRs） |
| 最佳场景 | 海量、非结构化、原始数据归档 | 高风险、权威的业务定义和规则 |

## 混合架构蓝图

OKF 并不会消灭 RAG——而是改变角色。AI Router 充当流量控制器：

```
[ User Request ]
           │
           ▼
    ┌──────────────┐
    │  AI Router   │
    └──────┬───────┘
           │
   ┌───────┴──────────────┐
   ▼                      ▼
[ OKF Bundle ]        [ RAG Pipeline ]
(Core Rules, Schemas, (Archived PDFs, Customer
 Runbooks, Precision)  Tickets, Scale Exploration)
```

OKF 提供确定性精度（核心规则、Schema、Runbooks），RAG 搜索广泛历史数据——构建既强大又稳定的 AI 系统。
