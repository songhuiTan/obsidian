# Hyper-Extract — 把文档抽成知识网：结构化知识抽取框架

> 别再切文档喂AI了！Hyper-Extract 一条命令把非结构化文本抽成知识图谱/超图/时间图，支持 Obsidian 导出和 MCP 调用

- **来源**：微信公众号 · 知识发电机（知识姬 Mina）
- **日期**：2026-06-26
- **标签**：`#RAG` `#知识图谱` `#LLM` `#结构化提取` `#知识管理`
- **GitHub**：[yifanfeng97/Hyper-Extract](https://github.com/yifanfeng97/Hyper-Extract) ⭐ 2.5k

---

## 痛点：传统分块检索的局限

传统 RAG 的 Chunking + Embedding 核心假设是"语义相近的块大概率包含答案"，但这个假设在需要明确 Schema 的场景里经常失效：

- 金融指标必须带**单位和时间戳**
- 法律条款必须有**义务主体和例外条件**
- 不同文档对同一实体的**命名不一致**（全称 vs 简称）→ Schema Drift

以前把精力放在 Chunk 策略和 Embedding 模型上，**提取阶段的结构化程度才是更上游的杠杆**。

---

## Hyper-Extract 是什么

LLM 驱动的**知识抽取与演化框架**。不是把文档切碎丢进向量库碰运气，而是用一条命令让 LLM 按模板把非结构化文本一次性抽成带类型的**结构化知识抽象（Knowledge Abstract）**。

### 核心理念

> **"构建一次，后面直接查结构"**

文档不再是临时上下文，而是变成**可持久、可演进的知识底座**。后续 agent 通过 MCP 直接查询结构化数据，不用每次都重跑 LLM 做信息抽取。

---

## 8 种知识结构

| 结构 | 说明 |
|------|------|
| **Model** | Pydantic 强类型数据模型 |
| **List / Set** | 简单集合 |
| **Graph** | 知识图谱（节点+边） |
| **Hypergraph** | 超图（一个超边同时连多个实体） |
| **Temporal Graph** | 带时间轴的图 |
| **Spatial Graph** | 空间图 |
| **Spatio-Temporal Graph** | 时空图 |

支持导出为 **Obsidian Vault**（带双链的 Markdown 文件），以及 **MCP-ready** 格式供 AI Agent 调用。

---

## 技术亮点

### 10+ 抽取引擎

| 引擎 | 定位 |
|------|------|
| **GraphRAG** | 微软开源，RAG + 知识图谱 |
| **LightRAG** | 轻量级图增强检索 |
| **Hyper-RAG** | 超图增强检索 |
| **KG-Gen** | 知识图谱生成 |

### 80+ YAML 模板

覆盖金融、法律、医疗、行业和通用场景：
- **金融**：公司实体、财务指标、风险点、时间关联
- **法律**：条款、主体、义务、例外条件
- **医疗**：病症、药物、治疗方案

### 增量演化

文档变化时知识结构可增量更新，不用全量重抽。不是静态快照。

### 本地运行

支持 vLLM 本地推理，数据不离开机器 → 处理敏感合同/内部财报时明确边界。

---

## 安装与使用

```bash
# 安装
uv tool install hyperextract

# 初始化 API Key
he config init -k YOUR_API_KEY

# 解析文档 → 知识图谱
he parse examples/tesla.md -t general/biography_graph -o ./output/ -l en

# 搜索结构化结果
he search ./output/ "What are Tesla's major achievements?"

# 查看知识结构
he show ./output/

# 导出为 Obsidian Vault
he export obsidian ./output/ -o ./vault/
```

---

## 典型场景

| 场景 | 效果 |
|------|------|
| 📄 **论文 → 研究图谱** | 追踪概念演变和引用关系 |
| 📊 **财报 → 公司-指标-风险图** | 后续自动化分析 |
| 🔒 **私有文档 → 可搜索库** | 本地运行，数据不外泄 |
| 🤖 **桌面 Agent 知识底座** | MCP-ready，结构化内存 |

---

## 点评

Hyper-Extract 的思路不是简单升级 RAG 工具，而是**给整个知识提取流程加了脊柱**：

- ✅ 把"每次都得重读"变成"构建一次，直接查结构"
- ✅ 骨架比碎片可靠 → 后续查询的幻觉和关系错误自然少一半
- ✅ 80+ 模板覆盖主流领域，开箱即用
- ✅ 增量演化支持文档变化，不是静态快照
- ⚠️ 模板太松 LLM 可能自由发挥，太严可能漏抽（需要迭代）
- ⚠️ 扫描件多时抽取完整度可能受影响

对于已经在用 Obsidian 整理笔记或跑 Agent workflow 的人，挑一个文档类型跑一遍模板，就能省掉反复问 AI "这个和那个的关系是？"的时间。
