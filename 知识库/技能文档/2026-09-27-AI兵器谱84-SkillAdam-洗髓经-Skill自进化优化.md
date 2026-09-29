---
tags: [AI, SkillAdam, Skill进化, Agent, ClaudeCode, Codex, RUC, 腾讯, 视频归档]
---

# AI兵器谱第84期：SkillAdam（Skill 洗髓经）

> 来源：AI架构前哨｜抖音 2026-09-27 08:15
> 视频：https://www.douyin.com/video/7690002074473434409
> GitHub：[ruc-datalab/SkillAdam](https://github.com/ruc-datalab/SkillAdam)（79⭐ / 3 forks / Python / MIT / 2026-09-06创建，09-26仍在更新）
> 论文：[SkillAdam: Stable and Efficient Skill Evolution for Agents](https://arxiv.org/abs/2609.08944)（arXiv 2609.08944，人大 RUC DataLab + 腾讯）
> 标签：AI, SkillAdam, Skill进化, Agent, ClaudeCode, Codex, 视频归档

---

## 概述

AI兵器谱第84期介绍 **SkillAdam**，江湖人称"洗髓经"——给现有 SKILL.md 加一层**测试 → 学习 → 受控改写 → 验证**的闭环，让 Skill 可以被持续优化而不是写完就死。

一句话：**把 Adam 优化器的两个动量思想搬到离散的 Skill 文档上，做 Skill 的自动进化。**

出品方：中国人民大学 RUC DataLab（Ju Fan 组）+ 腾讯。MIT 协议，有论文背书（7 个 benchmark SOTA）。

## 核心机制（Adam 类比）

| Adam 优化器 | SkillAdam 对应物 | 作用 |
|---|---|---|
| 一阶动量（梯度方向累积） | **Optimization Memory**：记录发现的问题 + 已尝试方案的成败 | 让有效的修正被累积，不被单次迭代的局部反馈覆盖 |
| 二阶动量（自适应步长） | **Volatility-driven Edit Budget**：按近期 case 级改进的一致性控制改动幅度 | 改进一致→大步改；波动大→小步改，防止技能被改崩 |
| 参数更新 | 受控的 SKILL.md 编辑 | 每次改动过 acceptance gate 验证，不通过不落盘 |

论文识别的两个挑战（也是卖点）：
1. **Direction Stability（方向稳定性）**：有效修正要能累积，不能被 iteration-local 反馈反复覆盖
2. **Update Adaptivity（更新自适应性）**：每次修订的范围要反映近期改进的一致性

## 产品形态

- **支持平台**：Codex · Claude Code · Cursor Agent · GitHub Copilot（每个平台一个 install.sh，注册 `skilladam-optimize` 技能）
- **用法**：在 agent 里说 "Use SkillAdam to optimize /path/to/SKILL.md for ..."，它会：
  1. 准备相关任务和评分规则
  2. 用当前 skill 跑测试（task rollouts）
  3. 提出修改 → 评估修改版 → 只有通过验证才写回文件
- **默认自动应用，可要求人工审查**："Before applying changes, show me the proposed edits and let me choose."
- **零额外基建**：用你已登录的 agent CLI（codex/claude/cursor-agent/copilot），无需本地 GPU/Docker/单独 API Key；模型调用走平台账号配额
- 一次只优化一个 SKILL.md；测试时 skill 资源和 workspace 文件不自动可见，上下文要写进任务里

## 代码结构（448 文件）

- `skilladam/core/`：acceptance_gate、edit_budget、momentum、feedback_loop、case_sampler、trajectory_condenser、type_taxonomy —— 核心闭环
- `skilladam/methods/`：skilladam.py + **skillopt.py**（对比基线，SkillOpt）
- `skilladam/backends/`：openai_compatible 等，DeepPlanning 桥接
- `skilladam/product/`：产品化运行时（jobs、session_store、patch_review、task_manifest 等 30 文件）
- `skilladam/cost/`：成本台账 ledger + pricing + report
- `skilladam/baselines/`、`benchmarks/`、`experiments/`：7 个基准的复现实验框架
- 每个平台集成都是一个 `SKILL.md`（integrations/*/skilladam/skills/skilladam-optimize/）

## 战略分析

### 与知识库现有体系的对照

| 维度 | SkillAdam | 知识库相关归档 | 说明 |
|---|---|---|---|
| 定位 | SKILL.md 自动优化闭环 | [[技能文档/2026-09-24-SkillRecorder又更新了-微软录屏转技能v0.5到v0.7]] 录屏→生成 | 一个管"改"，一个管"生" |
| 优化思想 | Adam 动量类比 | [[技能文档/2026-08-18-Yao-Meta-Skill-Skill-OS-技能工厂与治理系统]] 工厂+治理 | SkillAdam 是研究驱动的单点突破 |
| 验证门 | acceptance gate 落盘前验证 | [[技能文档/2026-09-05-从OpenSpec到AIDLC团队AI代码质量实践]] AIDLC 六大失败模式 | 都在解决"改了别改坏" |

### 值得借鉴的点

1. **"Skill 不是写完就结束"** 与知识库的 Skill 治理理念一致——SKILL.md 应该可回归、可审计、可持续进化。本知识库 340+ 技能文档本身就是"技能资产"，SkillAdam 的测试-验证闭环是缺失的一环
2. **Adam 二阶动量 → 编辑预算** 是最漂亮的类比：改动幅度随近期信号一致性自适应。这个思路对任何"LLM 自改文档"场景通用（比如自改 prompt、自改 wiki）
3. **验证门 + 人工可选审查** 的产品设计平衡：默认自动，可一键切人工选每个 edit——这是"自主性梯度"的好范本，和 ADK 的 Spectrum of Control 同源
4. **学术出身 + 工程落地** 双轨：论文 7 benchmark SOTA + 四平台 installer，RUC DataLab + 腾讯背景，比纯 demo 项目可信度高

### 风险与边界

- 79⭐ 早期项目，创建仅 3 周（2026-09-06），社区验证不足
- 测试信号来自 LLM 判分 + 平台配额消耗，优化一次有真实成本
- 与 Claude Code 原生 skill evolve 能力（如 Anthropic 官方方向）可能撞车，观察其差异化

## 归档日志

- 2026-09-27 归档（抖音视频页免登录提取元数据；GitHub API + arXiv 摘要 + 仓库树校验：79⭐、MIT、Python、09-26 仍在更新）
