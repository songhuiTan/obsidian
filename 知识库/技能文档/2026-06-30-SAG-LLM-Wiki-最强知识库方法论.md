---
title: "SAG+LLM WIKI：我将称之为最强知识库！"
author: "隐曜yinyo杂货铺"
source_url: "https://mp.weixin.qq.com/s/cpv8FOCZ02UhiMuyyu3EkA"
date: "2026-06-30"
tags: [SAG, LLM Wiki, 知识库, RAG, GraphRAG, 组织记忆, 企业知识管理, SQL-RAG, 多跳检索, Karpathy]
---

# SAG+LLM WIKI：我将称之为最强知识库！

> 很多公司建知识库，一不小心就建成了一座特别漂亮的网盘。界面很 AI，入口很现代，搜索框也很聪明。可一到真实业务问题，气氛就开始变得尴尬：资料能找回来，关系接不上；答案能生成出来，知识留不下来。
>
> 真正缺的不是文档数量，而是**组织记忆**。

企业知识库的**第一性问题**：**怎么让一个组织的经验，可以被找到、被连接、被更新、被质疑。**

## 知识库的三层能力

| 问题 | 需要什么能力 | 典型方法 | 
|:--|:--|:--|
| 找事实 | 相似召回、关键词召回 | RAG |
| 连关系 | 多跳事件链、实体扩展 | SAG / GraphRAG |
| 保可信 | 页面维护、证据回溯、持续更新 | LLM Wiki |

## RAG 建立了旧秩序

RAG 的核心动作是**相似召回**：文档切成 chunk → 转成 embedding → 用户提问时找最近片段 → 交给 LLM 回答。

对大量文档问答（如"报销流程需要哪些材料？"）够好。但面对**多跳链路问题**（如"华东门店预测达成率下降，和哪个供应商异常有关？"），普通 RAG 会把几段相关材料找回来并排躺着，但**穿不起那条因果链**。

## GraphRAG 的野心和代价

GraphRAG 很自然：把知识建成图（人/组织/商品/指标/系统 → 节点；影响/依赖/计算自 → 边），查询时沿图走。

**代价：** 需要持续治理本体论、实体归一、关系定义、别名处理。企业系统变化快，图谱跟不上就会变成"旧朝档案，格式还在，权力没了"。

**核心问题：** 为了让知识库具备关系检索能力，我们一定要先维护一张重型全局图吗？

## SAG 不修大城，它先追线索

**SAG**（SQL-Retrieval Augmented Generation）来自 arXiv 2606.15971，2026-06-14 提交。

核心思路：用 **event/entity + SQL join**，在**查询时动态构造局部关系结构**，减少对全局静态图的预构建依赖。

### 工作原理

1. 每段文本整理成 **event（事件）+ entities（实体）**
2. 存入 `event`、`entity`、`event_entity` 三张关系表
3. 查询时先找相关事件，再通过共享实体继续扩展（一步步沿着实体链走）

例如文本"6月18日，供应商A延迟交付SKU X，影响华东门店补货，预测达成率下降"：
- event：供应商A延迟交付SKU X → 影响华东门店补货 → 导致预测达成率下降
- entities：6月18日、供应商A、SKU X、华东门店、补货、预测达成率、延迟交付

查询时：从"预测达成率"找到事件A → 从事件A找到"SKU X" → 再通过"SKU X"找到事件B → 从事件B找到"供应商A"。

### 对比

| 方案 | 策略 | 维护成本 | 
|:--|:--|:--|
| RAG | 相似召回 | 低，但多跳弱 |
| GraphRAG | 全局静态图 | 高（持续治理） |
| **SAG** | **查询时动态组装局部关系链** | **低，天然支持增量写入** |

论文报告在 HotpotQA、2WikiMultiHop、MuSiQue 三个多跳 benchmark 上，9 个 Recall@K 指标里 8 个最好，MuSiQue Recall@5 为 80.0%。

## LLM Wiki：在 Agent 视角下对知识的长期维护

Karpathy 于 2026-04-04 发布 LLM Wiki gist。核心想法：**不要只让大模型在提问时临时检索；让大模型在资料进入时，就把资料整理成一套可读、可链接、可维护的 Wiki。**

三层结构：
- **raw sources** — 保存原始资料，作为证据源
- **wiki pages** — LLM 生成和维护的知识页面
- **schema / rules** — 约束 LLM 怎么组织、引用和更新

资料进来后，LLM 判断影响哪些页面（指标页、数据表页、系统页、异常事件页、专题综述页），新资料与旧页面冲突要标出，反复出现的概念要独立成页。

**关键约束：** Wiki 页面是生成层，不能当成最终事实源。每个判断必须能回溯到 raw source。

知识库的受众不再是人，而是 **Agent**。LLM Wiki 的核心就是让 Agent 更容易理解召回的知识库内容。

## 为什么是 SAG + LLM Wiki

企业知识库最容易死在**关系追不到**和**结论留不住**。SAG 补前者，LLM Wiki 补后者。

**组合工作流：**

1. 资料进入 → LLM Wiki 生成/更新知识页 + SAG 抽取 event/entities
2. 用户提问 → 先查 Wiki 页面获得整理过的上下文
3. → 再用 SAG 追原始事件链
4. → 回到 raw sources 校验证据
5. → 高价值答案写回 Wiki

## MVP 落地方案

三步走，不贪大：

| 层级 | 做什么 | 方法 |
|:--|:--|:--|
| 原始证据层 | 保存 PDF、会议纪要、SQL、DDL、看板说明 | raw sources，统一来源/版本/时间/引用路径 |
| 知识维护层 | 生成指标页、表页、异常页、专题页 | LLM Wiki，先做 20 个高频页面 |
| 深召回层 | 抽 event/entities，支持多跳追踪 | SAG，先覆盖指标/表/字段/任务/异常事件 |

**底线：** 定期 lint — 检查断链页面、无来源结论、过期指标口径、页面与原始 DDL 不一致。

## 适用场景

尤其适合数据产品：指标口径问答、数据表和字段血缘、供应链异常追踪、项目会议记忆、投研资料整理。

**不适合的情况：**
- 知识关系已高度标准化（如药品关系、法规条款）→ GraphRAG 更好
- 事件很少，长篇制度/手册/FAQ 为主 → 普通 RAG + 关键词检索足够
- 没有证据回溯习惯 → LLM Wiki 反而会成为幻觉放大器

## 参考引用

- SAG 论文：Yuchao Wu 等，《SAG: SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges》，arXiv:2606.15971，2026-06-14. [https://arxiv.org/abs/2606.15971](https://arxiv.org/abs/2606.15971)
- Karpathy LLM Wiki gist：`llm-wiki.md`，2026-04-04. [https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- nashsu/llm_wiki 项目：[https://github.com/nashsu/llm_wiki](https://github.com/nashsu/llm_wiki)
