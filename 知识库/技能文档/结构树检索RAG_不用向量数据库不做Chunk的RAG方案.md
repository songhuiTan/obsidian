---
tags:
  - RAG
  - 检索增强生成
  - 结构树检索
  - PageIndex
  - AI架构
source: 抖音@北山AI说 / zhuanlan.zhihu.com / 53ai.com
date: 2026-07-12
status: completed
author: 听风吟者李狗嗨（知乎）
---

# 结构树检索RAG：不用向量数据库、不做Chunk的RAG方案

## 核心洞察

**相似性 ≠ 相关性。** 对于结构化专业文档，「哪段文字像问题」和「答案在哪」是两个问题。后者要推理，不是搜。

向量RAG最大痛点：语义上像的段落，不一定包含答案。财报问「盈利能力」，向量空间里「盈利」和「市场份额」挨得太近，返回的是错误内容。

## PageIndex — 31K Star 扛旗项目

- **GitHub:** VectifyAI/PageIndex（MIT协议）
- **准确率:** FinanceBench **98.7%** vs 传统向量RAG 70-80%
- **代价:** 延迟3-8秒（vs 50-150ms），成本高5-15倍
- **基础设施:** 只需一个JSON文件（vs 向量DB + embedding管线）
- **可解释性:** 完整推理链 + 页码

### 工作原理

LLM 提取 PDF 目录页 → 生成 JSON 树结构 → 查询时两次 LLM 调用：
1. 在树结构上导航定位节点
2. 从定位到的节点原文出答案

**三件事PageIndex能做到而向量RAG做不到：**
- 跟踪交叉引用：「详见附录G」直接跳G节点
- 并行查多章节：同时返回多个 node_id
- 理解领域同义词：「盈利能力」识别「EBITDA margins」

### 快速上手
```bash
pip3 install --upgrade -r requirements.txt
# 设置 CHATGPT_API_KEY 环境变量
python3 run_pageindex.py --pdf_path /path/to/document.pdf
# 或处理 Markdown
python3 run_pageindex.py --md_path /path/to/doc.md
```

---

## 结构树检索全景（四类方案）

### A类：用文档原本的结构

核心思路：直接读目录/标题层级，提取成树。

| 方案 | 特点 | 指标 |
|------|------|------|
| **PageIndex** | 31K★，提取目录为JSON树 | FinanceBench 98.7% |
| **TreeDex** | PDF结构自动发现，字体分析打标题标记 | — |
| **TreeSearch** (shibing624) | 190★，SQLite FTS5 + 结构感知，毫秒级零成本 | 不调LLM |

### B类：LLM自己建语义树

文档原生结构不够时，让LLM递归建树。

| 方案 | 来源 | 核心方法 |
|------|------|----------|
| **MemWalker** | Meta FAIR (2023) | 切chunk → LLM逐层摘要合并自底向上建树；从根出发导航到叶子 |
| **LATTICE** | ICLR 2026 | 日志复杂度导航，校准分数从根到节点聚合，按深度衰减。BRIGHT基准42万篇零样本nDCG@10 = 51.6 |
| **FABLE** | 2026.1 | LLM + 向量双路并行，预算路由控制token。省94% token（31K vs 517K），答案完整性92.07% |
| **ToM** | EMNLP 2025 | 在树上递归推理不是检索后生成。70B级模型上超过RAG和DCF |

### C类：Agent先定位再精读

| 方案 | 核心方法 | 指标 |
|------|----------|------|
| **DeepRead** | 三维坐标 {doc_id, sec_id, para_idx} + Retrieve/ReadSection工具 | 4基准平均比Agentic搜索高10.3% |
| **RDR²** | 三个原子动作 [ANS/EXP/REF]，路由可训练 | 5数据集SOTA |
| **BookRAG** | 树+图融合，GT-Link映射实体到原文位置 | M3DocVQA EM提升18%，token省10x |

### D类：推理树/图智能体

| 方案 | 核心方法 |
|------|----------|
| **GraphReader** (EMNLP 2024) | 4K上下文窗口超过GPT-4-128K全文 |
| **HiRAG** | 530★，层级KG三级检索 |
| **RT-RAG** (2026.1) | 多跳推理树，F1 +7% |
| **ReTreever** (ServiceNow) | 可学习树，帕累托最优 |

---

## 方案选择速查

| 需求 | 推荐方案 | 核心优势 |
|------|----------|----------|
| 最高精度 | PageIndex | 98.7%，代价是慢且贵 |
| 最快/最小开销 | TreeSearch / ReTreever | 毫秒级，零API成本 |
| 超大规模零样本 | LATTICE | 面向42万+文档 |
| 精度+成本兼顾 | FABLE | 双路检索+预算控制 |
| 多跳推理 | ToM / RT-RAG | 树状MapReduce |
| 书/复杂文档 | BookRAG / HiRAG | 树图融合更稳 |

## 发展方向

1. **树和图的融合** — 树管导航，图管实体推理，映射层是桥
2. **路由从手工走向学习** — RDR²证明路由可训练，ReTreever证明整棵树可学习
3. **零LLM分支** — TreeSearch和ReTreever代表极高效率极低成本，适合嵌入设备、离线场景

## 最终判断

- **结构化文档场景**（财报、法律文书、技术手册）：结构树导航为主力，向量库降级
- **非结构化场景**（客服对话、社交媒体、用户笔记）：向量RAG仍是首选

> 以后的生产RAG架构，大概率不是选哪个方案的问题，而是怎么判断一个查询该走哪条路的问题。

---

## 参考
- 抖音：北山AI说《不用向量数据库不做Chunk,RAG还能怎么玩》
- 知乎：听风吟者李狗嗨《扔掉向量数据库，RAG还能跑吗？结构树检索深度综述》
- 53AI：《不用向量数据库的RAG，居然跑得更准了？》
- PageIndex GitHub: github.com/VectifyAI/PageIndex (31K★)
