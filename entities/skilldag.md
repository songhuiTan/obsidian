---
title: SkillDAG
created: 2026-06-04
updated: 2026-06-04
type: entity
tags: [skill, agent, architecture, open-source]
sources: [知识库/技能文档/2026-06-04-SkillDAG把孤立技能变成会进化的关系图.md]
confidence: high
---

# SkillDAG

SkillDAG 是复旦大学、新加坡国立大学（NUS）和 A*STAR 联合提出的技能关系图框架。它显式建模技能之间的依赖、冲突、特化等关系，让 Agent 在技能检索时能基于图结构做决策，而非仅靠 Embedding 相似度。

## 核心问题

传统技能检索（Embedding 余弦相似度 + Top-K）存在三个盲区：
- **前提技能被忽略**：互补技能在 Embedding 空间里距离很远，检索不到
- **冲突与冗余混淆**：互斥技能和替代技能在余弦分数上表现一致
- **库越大检索越差**：5000 份技能时 Top-5 容易被干扰项淹没

## 五种边类型

| 边类型 | 含义 | Agent 行为 |
|--------|------|-----------|
| depends_on(A, B) | A 需要 B 为前提 | 选 A 必须同时加载 B |
| specializes(D, A) | D 是 A 的特化版本 | 领域匹配时优先选 D |
| composes_with | 组合使用效果更好 | 提示可一起加载 |
| similar_to | 功能重复可互换 | 任选其一 |
| conflicts_with | 一起用会导致失败 | 选 A 排除 B |

## Agent 可调用的图接口

- **search(query, K, D)** — 同时返回 matches（余弦相似）、neighbors（图遍历）、conflicts（冲突检测），三个信号分开不融合
- **show(skill)** — 按需加载技能完整内容，不提前灌满上下文

## 冷启动与在线演化

- **两视图建图**：e_self（技能自我描述）+ e_needs（技能前置条件），互补视图捕捉依赖关系
- **在线演化**：Agent 执行中可调用 `propose-edge`（预览）和 `edit-edge`（提交）来动态修改图关系

## 与相关方法对比

- **GoS (Graph of Skills)**：离线建图 + 固定 PageRank 扩散，Agent 不知图结构
- **SkillDAG**：图是 Agent 的工具，在线可查可改，LLM 自主决策

## 相关页面

- [[openclaw]] — Skill 体系可借 SkillDAG 优化检索
- [[skill-development]] — OpenClaw Skill 开发方法
