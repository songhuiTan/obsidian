---
title: "阿里 SkillWeaver：跳过加载全部工具，Agent Token 消耗直降 99%"
source: "何码先生"
source_url: "https://mp.weixin.qq.com/s/ZMHTVLsarxvQGA8xorLoOA"
date: "2026-07-06"
tags: [AI, Agent, MCP, SkillWeaver, 阿里, 工具路由, SAD, 论文, FAISS]
---

## 概述

阿里巴巴研究院提出的 **SkillWeaver** 框架，解决 Agent 在海量工具（MCP 生态成百上千个工具）下的高效路由问题。核心思路是"不念菜单，按需上菜"——通过三阶段管线（Decompose→Retrieve→Compose）替代传统的暴力塞入方式。在 2209 个真实 MCP 技能的基准测试中，Token 消耗从 884,000 降到约 1,160（降幅 99.9%），同时准确率大幅提升。论文已发布在 arXiv（2606.18051）。

## 核心问题：暴力塞入的死胡同

当前主流 Agent 工具调用方式是把所有工具描述塞进 LLM 的上下文窗口。当企业接入成百上千个 MCP 工具后：

| 问题 | 数据 |
|------|------|
| Token 爆炸 | 2209 个技能描述 → 单次查询 ~884,000 Token |
| 选择混乱 | Qwen-Max 在 2209 个工具前正确检索率仅 21.1% |
| 上下文溢出 | 绝大多数 LLM 窗口装不下这么多工具描述 |

## 核心架构：三阶段管线

### 1. Decompose（任务分解）
将用户查询拆解为原子子任务。例如"下载数据集、转换格式、生成可视化报告"拆为三个子任务。

### 2. Retrieve（技能检索）
对每个子任务语义匹配 Top-K 候选工具。使用 all-MiniLM-L6-v2 + FAISS 索引，2209 个技能建库仅需 15 秒，检索延迟低于 15 毫秒，LLM 不参与此次计算（0 Token）。

### 3. Compose（计划组合）
评估候选工具兼容性，生成 DAG 执行图（有向无环图），标注依赖关系和并行机会。

## 核心创新：SAD 反馈环路

**迭代式技能感知分解（Skill-Aware Decomposition, SAD）** 是最核心的创新。解决的是"词汇不匹配"问题：

1. LLM 分解 → 倾向用通用模糊描述（"获取数据"）
2. 语义检索 → 找到工具库中的特定术语（"api-client"、"http-fetch"）
3. 回注工具描述 → 告诉 LLM"库里有 api-client"
4. LLM 重写 → "用 api-client 获取"

循环直到子任务与工具库词汇对齐。

**SAD 效果：**

| 模型 | 无 SAD | + SAD |
|------|--------|-------|
| Qwen2.5-7B | 51.0% | 67.7%（↑33%） |
| Qwen-Max | — | 92% |
| 困难任务（4-5 技能协作） | — | ↑50% |

## 反直觉发现：更大的模型可能更差

14B 模型无 SAD 时的分解准确率 **低于** 7B 模型。原因是更大的模型倾向于过度分解——把一步操作拆成四五步微观子任务，这些微观步骤在技能库中找不到匹配工具。SAD 的回注机制恰好解决这个问题：检索到的真实工具描述像"锚"一样把分解粒度拉回现实。

> "Aligning an agent with the vocabulary of specific tools is often more impactful than paying for a larger, more expensive LLM."

## Token 消耗拆解

| 方法 | Token 消耗 | 准确率 |
|------|-----------|--------|
| 暴力塞入（LLM-Direct） | ~884,000 | 21.1% |
| SkillWeaver（精准路由） | ~1,160 | 大幅领先 |

核心差异：暴力塞入让 LLM 读整本百科全书再回答，SkillWeaver 先用向量检索翻到正确页码，再让 LLM 只读那一页。

## ReAct 的彻底失败

传统 ReAct 风格 Agent 在 CompSkillBench 上的分解准确率为 **0%**——不是低，是彻底失败。ReAct 是反应式（走一步看一步），无法提前规划多工具序列。SkillWeaver 是规划式（先分解→检索→组合，一步到位生成完整 DAG）。

## 实现路径

论文使用的全部是现成组件，未开源但可自行复现：

| 组件 | 方案 |
|------|------|
| 任务分解 | Qwen2.5-7B / 任意 LLM + Prompt 模板 |
| 语义检索 | all-MiniLM-L6-v2 + FAISS（可升级 BGE-base-en-v1.5） |
| SAD 环路 | Prompt Engineering + 检索反馈（论文已给模板） |
| 编排框架 | LangChain / LlamaIndex / 原生 Python |
| 重排序 | Cross-Encoder 或 LLM-based Reranker（可选增强） |

**生产注意事项：** Bi-Encoder 将正确工具召回 Top-10 的概率约 70%，但排第 1 的概率仅约 37%。生产环境大概率需要加一层 Cross-Encoder 或 LLM Reranker 重排序。

## 尚未解决的问题

SkillWeaver 聚焦路由和规划，但以下问题仍是开放题：

- **错误恢复** — DAG 执行到某步 API 超时/鉴权失败，整个链条断裂
- **超时重试** — 对每个节点设置重试策略
- **降级方案** — 首选工具失败时自动切到备选
- **断点续传** — 部分节点失败后已成功的不需重跑
- **输出校验** — 每一步输出格式校验

## 关键洞察

### 三层启发

1. **认知层**：对齐 > 算力。7B + SAD 打败裸奔 14B。ReAct 0% 揭示反应式架构的结构性缺陷。
2. **架构层**：分解粒度是瓶颈，SAD 反馈环路是解法。"规划的质量决定一切下游的上限"。
3. **生态层**：MCP 索引层将从可选优化变成必需基础设施——"谁先在 MCP 生态建好索引层，谁就掌握 Agent 工具调用的入口"。

### 范式转变
- 从"选一个工具" → "组合一条工作流"
- 从"一次决策" → "迭代对齐"
- 99.9% Token 降幅是这个范式转换的自然结果

## 战略分析

**与 Hermes Agent 体系的对照：**

SkillWeaver 直接触及 Hermes 的核心运作模式——工具/技能路由。Hermes 当前通过 skills/ 目录 + skill_manage 管理技能，技能数量在数十级别。如果技能规模膨胀到数百（集成大量 MCP 工具），SkillWeaver 的路由思路就直接相关。

**差距与借鉴点：**

1. **Hermes 当前的路由模式**：Hermes 的 skill 系统本质上是显式加载——用户指定 skill 名称，引擎读取 SKILL.md。这和"暴力塞入"不同（不塞全部，只塞选中的），但也缺乏 SkillWeaver 的语义检索 + 组合式 DAG 规划能力。

2. **SAD 反馈环路**：让 LLM 学会用"工具的语言"说话——这个思路对 Hermes 的 function calling 优化有直接参考价值。当 Hermes 对接大量 MCP 工具时，工具描述与 LLM 意图之间的词汇对齐是普遍问题。

3. **ReAct 0% 的警示**：Hermes 的子代理执行模式本质上是 ReAct-like（走一步看一步）。对于需要多工具协作的复杂任务，这种模式可能不是最优的。SkillWeaver 的规划式（Decompose→Retrieve→Compose）可能更适用于多步工作流。

4. **Token 优化**：Hermes 的每个 tool call 都会把工具描述注入上下文。如果集成大量 MCP 工具，这部分的 Token 消耗需要纳入考虑。

**整合可能性：中高。** SkillWeaver 的三阶段管线可以嵌入 Hermes 的 skill 系统作为"技能发现层"——用户提出目标，系统自动 Decompose→Retrieve→Compose，生成 DAG 执行计划后由 Hermes 引擎执行。这与 Hermes 当前的手动 skill 加载模式互补，不冲突。

特别是 FAISS 索引层 + SAD 环路可以在 Hermes 的 skill hub 上实现——对 ~/.hermes/skills/ 的所有 SKILL.md 建索引，实现语义化的技能发现而非仅靠名称匹配。

## 相关资源

- 论文：SkillWeaver: Compositional Skill Routing for LLM Agents — arXiv 2606.18051
- VentureBeat 报道：https://venturebeat.com/orchestration/new-alibaba-ai-framework-skips-loading-every-tool-cutting-agent-token-use-99
- all-MiniLM-L6-v2：https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2
- BGE-base-en-v1.5：https://huggingface.co/BAAI/bge-base-en-v1.5

## 归档日志

- 2026-07-18 归档
