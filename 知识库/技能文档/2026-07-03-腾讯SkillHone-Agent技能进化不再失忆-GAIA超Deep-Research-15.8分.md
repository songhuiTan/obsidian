---
title: "腾讯开源SkillHone：Agent技能进化不再「失忆」，GAIA超商用Deep Research 15.8分"
source: "Hyman的杂货铺"
source_url: "https://mp.weixin.qq.com/s/2GJgjLP1lelYIJ_58GYiFw"
date: "2026-07-03"
tags: [AI, Agent, Skill, SkillHone, 腾讯, 微信, 论文, 决策历史, 开源]
---

## 概述

腾讯微信团队提出 **SkillHone** 框架，解决 Agent Skill 维护中的核心问题：**技能进化不仅要产出更好的 Skill 包，也要留下可跨会话复用的决策历史**。在 GAIA 和 WebWalkerQA 上分别比商用 Deep Research Agent 高出 15.8 和 3.2 个百分点。论文已开源到 GitHub。

> 论文：SkillHone: A Harness for Continual Agent Skill Evolution Through Persistent Decision History
> arXiv：2606.08671 | GitHub：github.com/Tencent/SkillHone

## 核心问题：技能进化的「失忆」困境

当 Skill 需要跨会话持续维护时（API 变更、限流、页面结构漂移），后续 Agent 只拿到最新的 SKILL.md 快照，看不到：
- 上一轮诊断出了什么
- 哪些补丁被拒绝过
- 评估探针暴露了哪种失败模式
- 为什么最终选了 A 而不是 B

结果：Agent 可能重复第一轮已经试过、后来证明会拖慢流程的方案。

## 三类现有路线的局限

| 路线 | 代表 | 局限 |
|------|------|------|
| 合成式构建 | Skill-Creator | 适合冷启动，但只留最终产物 |
| 反射式优化 | Hermes-SE / GEPA | 单轮有效，轮次结束后历史不保留 |
| **SkillHone** | 本文 | 跨会话持续维护，保留完整决策链 |

## SkillHone 核心设计：双仓库 + 角色隔离

### 决策历史四元组

每一轮优化记录 `<d, r, e, o>`：

- **d**（诊断）：当前失败模式是什么
- **r**（修订）：打算怎么改 Skill
- **e**（证据）：脱敏后的评估反馈摘要
- **o**（决策）：接受、拒绝、要求再改、暂缓

### 两个关联仓库

- **Skill 仓库**：存 SKILL.md、脚本、参考文档、模板
- **Skill-Eval 仓库**：存练习探针、标准目标、验证器、轨迹、脱敏报告

### 角色隔离

| 角色 | 所属团队 | 能做什么 | 不能做什么 |
|------|---------|---------|----------|
| 诊断员 | 优化 | 读脱敏报告+历史、写诊断 | 访问未脱敏探针 |
| 提案员 | 优化 | 起草 Skill 修订、开 PR | 访问验证器/轨迹 |
| 探索员 | 优化 | 整合既往资源、读失败维基 | 写评估仓库 |
| 执行员 | 评估 | 在探针上跑 Skill | 写 Skill 仓库 |
| 分析员 | 评估 | 分析轨迹、定位失败模式 | 写 Skill 仓库 |
| 报告员 | 评估 | 输出脱敏问题报告 | 写 Skill 仓库 |

**关键约束**：优化侧只能看到脱敏报告，看不到未脱敏的目标、验证器或完整轨迹——防止 Agent 在探针上背答案。

### GitHub 式工作流

- **Issue**：记录诊断出的失败模式
- **PR**：承载 Skill 修订提案
- **Merge/Reject**：记录最终决策

## 实验主结果

### GAIA（裸网公开网页）

| 系统 | L1 | L2 | L3 | 平均 |
|------|-----|-----|-----|------|
| Deep Research Agent（商业搜索） | 61.9 | 47.0 | 26.3 | **48.8** |
| Existing-Skills | 64.3 | 33.3 | 21.1 | 41.7 |
| Skill-Creator | 64.3 | 37.9 | 21.1 | 44.1 |
| Hermes-SE | 73.8 | 40.9 | 31.6 | 50.4 |
| **SkillHone** | **76.2** | **66.7** | 31.6 | **64.6** |

SkillHone 比带商业搜索的 Deep Research Agent 高 **15.8 点**，L2 从 40.9% 拉到 66.7%。

### WebWalkerQA-EN

| 系统 | Easy | Med. | Hard | 平均 |
|------|------|------|------|------|
| Deep Research Agent | 58.5 | 62.3 | 67.1 | **63.2** |
| SkillHone | 53.7 | **69.2** | **68.4** | **66.4** |

### 跨模型迁移

同一套 Skill 包直接换 Claude Sonnet 4.6 执行（不做额外优化）：
- SkillHone：**72.4%**
- Hermes-SE：62.2%
- Existing-Skills：56.7%

提升来自 **Skill 流程本身**，换骨干模型不需要重新优化。

### 消融实验

| 变体 | GAIA | WebWalkerQA |
|------|------|------------|
| 完整 | 64.6 | 66.4 |
| 去掉决策历史 | 51.2（-13.4） | 55.5（-10.9） |
| 去掉角色隔离 | 58.2（-6.4） | 61.1（-5.3） |

决策历史贡献更大——跨会话可复用的"为什么这样改"比单轮权限切分更稀缺。

### 内部部署：7 个场景平均 +18.8 点

计数(+30.0)、聚合(+26.3)、结构解析(+25.0)、密度估计(+23.1)、跨度检索(+21.5)、过滤排序(+5.9)、列表过滤(+0.0)

## 五轮进化实录（Table 4）

| 轮次 | 诊断 | 修订 | 决策 |
|------|------|------|------|
| R1 | DuckDuckGo 超时频发 | 加重试 + wiki/wikidata 脚本 | 合并 |
| R2 | 真实网页搜索偶发失败 | ddg shell 换 Python + Jina | 合并 |
| R3 | 前两轮各有优点 | 精准合并 + 收紧预算 | 合并 |
| R4 | 名字格式解析失败 | Wikidata SPARQL + 收紧预算 | 合并 |
| R5 | 预算调高后分数下降 | **只回滚预算，保留 SPARQL 和名字规则** | 合并 |

第 5 轮是局部回滚的典型——无决策历史下不可能精准执行。

## 战略分析

**与 Hermes Agent 体系的直接相关度：最高。**

SkillHone 解决的正是 Hermes Agent 的 skill 维护问题。当前 Hermes 的 skill 管理是"最终产物模式"：用 `skill_manage` 创建/更新 SKILL.md，但缺少跨会话的决策历史。

**与已归档文章的呼应：**

| 文章 | 与 SkillHone 的关系 |
|------|-------------------|
| [[2026-06-15-小米HarnessX-Agent-Harness进化-弱模型最高+44%]] | HarnessX = 把 Harness 作为进化对象的框架；SkillHone = 把 Skill 进化过程本身作为管理对象的框架。互补：HarnessX 关注"改什么"，SkillHone 关注"怎么记改的过程" |
| [[2026-07-18-Lisp-Agent-OS-比Rust-Hermes更强大]] | Lisp Agent OS 用 Event Sourcing 记录运行时状态；SkillHone 用决策历史记录优化过程。一个记"运行痕迹"，一个记"改的理由" |
| [[2026-07-06-阿里SkillWeaver-跳过加载全部工具-Agent-Token直降99%]] | SkillWeaver 是静态路由；SkillHone 是跨会话 Skill 进化。不同维度 |

**最值得借鉴的设计：**

1. **GitHub 式 Issue/PR/History 工作流** — 这是最直接可用的模板。Hermes 的 skill 更新可以借鉴"每次修改都要关联 Issue（诊断）+ PR（变更）+ 决策结果"的模式。
2. **角色隔离 + 脱敏反馈** — 写 Skill 的 Agent 和测 Skill 的 Agent 分开。当前 Hermes 的 skill 测试是 agent 自测自评，没有这种权限隔离。
3. **局部回滚** — 当一次修订部分有效、部分有害时，有历史就能局部回滚；没有历史就只能全接受或全拒绝。这是决策历史最直接的实操价值。

**推荐的操作建议（来自论文）：**
1. 给 Skill 维护建评估仓库（至少一小套练习探针 + 验证器）
2. 记录"失败现象 → 假设原因 → 尝试修订 → 证据 → 决策"的 Issue 级诊断链
3. 权限隔离——写 Skill 的 Agent 和跑探针的 Agent 分开

**开源仓库已公开**：https://github.com/Tencent/SkillHone — 可以直接克隆参考。

## 相关资源

- 论文：arXiv 2606.08671
- GitHub：https://github.com/Tencent/SkillHone
- 项目页：https://zwlijay.github.io/SkillHone-Project
- 同系列文章（Hyman的杂货铺）：HarnessX、SkillHone 为同一作者的两篇深度拆解

## 归档日志

- 2026-07-18 归档
