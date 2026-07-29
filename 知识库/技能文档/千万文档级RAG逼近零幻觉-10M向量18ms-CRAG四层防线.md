---
tags:
  - RAG
  - 零幻觉
  - 大规模检索
  - Agentic RAG
  - CRAG
  - 引用验证
tags:
  - RAG
  - 零幻觉
  - 大规模检索
  - Agentic RAG
  - CRAG
  - 引用验证
source: 微信公众号 深度记事
author: codexlz
date: 2026-07-12
status: completed
---

# 千万文档级 RAG 如何逼近零幻觉

> 公众号 **深度记事** · codexlz · 2026年6月25日（归档重发）
> GitHub: [FareedKhan-dev/rag-zero-hallucinations](https://github.com/FareedKhan-dev/rag-zero-hallucinations)
> 全套代码 + Notebook，本地部署，单张 H100

## 核心思路

**不对抗模型"会猜"的天性，而是把系统设计成只有一种安全失败方式：拒答。**

四层控制机制：
1. **检索** — 混合稠密+BM25 + 上下文化 chunk + 重排
2. **约束生成** — 严格基于上下文，每句带引用，否则输出 abstain token
3. **验证闸门** — 每条原子级 claim 与引用文本逐一核验
4. **拒答决策** — 支持度低于校准阈值时直接拒绝回答问题

---

## 完整 Pipeline（10 个组件）

### 1. 数据准备
- HotpotQA（7,405 题，附 gold supporting facts）+ SQuAD v2 impossible questions + 手工 false-premise
- 固定随机种子，确保可复现

### 2. 清洗语料
- NFKC 标准化（fi 连字还原、空白折叠等）
- **MinHash LSH 近重复去重**（threshold=0.9）— 去掉 19 条近重复

### 3. 结构感知切块（Structure-Aware Chunker）
- 按句子边界打包，保留 token 预算内完整句子
- 加 overlap 避免答案被截断丢失
- 21,259 chunks，平均 125 tokens

### 4. 上下文化（Contextual Prefix）
- 每个 chunk 前补一句 LLM 生成的定位上下文
- "Before: Ed Wood is a 1994 American biographical... → After: This chunk introduces the 1994 film *Ed Wood*..."
- **最高性价比的召回提升手段**

### 5. 混合索引（Hybrid Index）
| 类型 | 技术 | 优势 |
|------|------|------|
| Dense | Qwen3-Embedding-4B → LanceDB | 语义改写匹配 |
| Sparse | BM25 + stemming | 精确命中名称/ID/数字 |
- RRF 融合（k=60），召回 150 个候选

### 6. 重排（Reranker）
- Qwen3-Reranker-4B cross-encoder
- 150 候选 → 精排保留 top 20
- Gold passage recall**0.97**（HotpotQA）

### 7. 路由与拆解
- Router: `no_retrieval` / `single_hop` / `multi_hop`
- Decomposer: 多跳问题拆为 2-3 个子问题
- False-premise 检测：判断查询是否有不成立的前提

### 8. 带引用生成
- System prompt 强制：**只能基于上下文，每句附带 passage id 引用**
- 引用过滤：模型伪造的 id 会直接被剥离
- Abstain token: `INSUFFICIENT_EVIDENCE`

### 9. 验证闸门（核心防线）
- **Claim 提取**：答案拆为原子级 claim（如 "Scott Derrickson is American"）
- **Faithfulness Judge**：Qwen3-32B 做 strict fact-checker（每条 claim vs 引用文本打分）
- **最弱 claim 规则**：min_support 低于阈值直接判败
- CoVe（Chain-of-Verification）：边缘答案给 1 次修改机会

### 10. 拒答决策（Abstention Policy）
多信号汇聚：router判定 + 模型 abstain + gate 结果 + 校准阈值
- **只输出 verified 状态或明确拒答**

---

## Agentic CRAG 循环（LangGraph）

```
route → retrieve → grade → (if OK) generate → verify → finalize
                          → (if weak) refine → retrieve (loop)
                          → (if hopeless) finalize (abstain)
```

- `grade_evidence` 评分 0-1
- OK ≥ 0.7 → generate | 0.4-0.7 → refine | < 0.4 → abstain
- 最多 3 次纠错 hop

---

## 评估结果

### 200 题 Golden Set（100 可回答 + 100 不可回答）

|  | 已回答 | 已弃答 |
|------|--------|--------|
| 可回答 | **46** | 54 |
| **不可回答** | **2（幻觉）** | **98** |

- **幻觉率：2%**（100 个不可回答问题只答了 2 个）
- **Faithfulness：0.908**（已回答问题上）
- Context Recall@20：**0.97**
- 可回答准确率：0.58（安全性代价：更倾向弃答）

### Scalability（10M+ 向量）

- LanceDB（嵌入式，落盘在 NVMe，不占 RAM）
- 100K → 1M → **10M 向量**
- **10M 向量查询延迟：18ms**
- 推算 100M：约 200ms

### Verifier 验证（HaluBench）
- AUROC：**0.702**（300 条人工标注样本）
- 有提升空间，架构支持直接替换 verifier

---

## 硬件与模型

| 资源 | 明细 |
|------|------|
| GPU | 单张 **NVIDIA H100 80GB** |
| 内存 | 180 GB |
| 磁盘 | 750 GB NVMe |
| Generator | **Qwen3-32B**（vLLM 独立进程，temperature=0） |
| Embedding | Qwen3-Embedding-4B |
| Reranker | Qwen3-Reranker-4B |
| Verifier | 复用 Qwen3-32B 通过 prompt |
| 峰值 VRAM | 62 GB / 80 GB |

全部模型本地部署，文档和查询不离开机器，适用于私有语料库。

---

## 核心设计哲学

> **与其追求一个完美模型，不如把一个普通模型包裹进一套系统里，让它只有一种安全的失败方式：拒答。**

- 可复现性优先：固定所有随机种子
- Claim-level verification，不是整段评分（最弱链条决定整体）
- 引用过滤：模型伪造的 citation id 自动剥离
- 弃答路径更快（3.3s vs 4.6s），因为不经过生成和验证
- 阈值可调：不同领域"答错代价"不同时可独立校准
