---
title: "我做了全球首个会「自进化」「自微调」的 Code Agent：momo Code（V1.0.0）"
author: "莫莫momozi (momo子讲AI)"
source_url: "https://mp.weixin.qq.com/s/MWW8eczRSylEAoRELntS7Q"
date: "2026-06-26"
tags: [momo Code, Code Agent, 自进化, 自微调, OpenCode, 贝叶斯, Thompson采样, 棘轮门控, 开源]
---

# 我做了全球首个会「自进化」「自微调」的 Code Agent：momo Code（V1.0.0）

> 模型是静态的。你教它一次，它记住了；你纠正它十次，它可能还会犯第十一次。就像一个从不记笔记的学生。每次对话都是从零开始，昨天的经验今天全忘。
>
> 如果 Code Agent 能像人一样——从每次交互中学习、积累经验、持续进化——会怎样？

**MOMO CODE** 是一个完全开源的、会自进化、会自微调的 AI 编程 Agent 产品。

- 官网：https://momozi.cc
- 开源地址：https://github.com/momozi1996/momo-code

## 核心设计：双速进化（Two-Speed Evolution）

灵感来自生物学：细菌有快速适应（应激反应）和长期进化（自然选择），两个时间尺度并行。

### 🔥 快环 /evolve —— 【自进化】秒级经验积累

每次完成一个任务，MOMO CODE 观察整个过程中的信号：
- 测试通过了？✅ 记一笔
- 编辑被你接受了？✅ 记一笔
- 编译报错了？❌ 记一笔
- 你手动纠正了它？❌ 记一笔

当同一个模式出现 **3 次以上**，系统自动提炼出一条经验策略（tactic）。下次做类似任务时，这些策略通过 **Thompson 采样**（一种贝叶斯方法）自动注入到系统提示里。

### 🧬 慢环 /fine-tune —— 【自微调】周期性能力跃迁

当经验足够多时触发完整训练管线：

```
课程合成(Curriculum) → 基线评估(Baseline) → 训练(Train) → 候选评估(Candidate) → 棘轮门控(Ratchet Gate) → 晋升(Promote)
```

**棘轮门控（Ratchet Gate）** 是核心机制：
```
PASS iff candidate.passAt1 >= baseline.passAt1 - 0.02 AND regressions == 0
```
候选模型必须在所有测试集上至少和基线一样好（允许2%噪声容差），且不能有任何回归。**只进不退**，旧版本自动备份，一键可回滚。

默认驱动器是 **Priors（贝叶斯先验更新）**，纯CPU、秒级完成。也可接入 PEFT/Transformers 做真正的 LoRA 微调。

## 算法架构：Bayesian + Thompson + Ratchet

1. **Beta(α, β) 贝叶斯追踪** — 每条策略维护 Beta 分布，α = 1 + wins，β = 1 + losses。胜率 = α/(α+β)。
2. **Thompson 采样** — 从每条策略的 Beta 分布中随机采样一个值，按采样值排序选取前6条注入。自然平衡「利用」和「探索」。
3. **Ratchet Gate（棘轮门控）** — 只进不退的晋升机制，防止模型退化。

## 系统架构：四层 + 25+ 模型

| 层级 | 说明 |
|:--|:--|
| **基础设施层** | 25+ LLM Provider Hub（DeepSeek/GLM/Kimi/豆包/MiniMax/Claude/GPT-4/Gemini/OpenRouter…），统一 OpenAI-compatible API |
| **Agent 核心层** | SSE 流式输出、系统提示动态组合（身份+安全+工具指引+注入策略）、会话管理、git 快照回滚 |
| **经验层** | 双速进化引擎 + 统一贝叶斯追踪存储（tactics.json + ledger.jsonl） |
| **UI 层** | ASCII 艺术横幅、彩色终端输出、`--json` 模式（CI 集成） |

## 关键特性

- 基于 **Opencode** 框架衍生
- 支持自定义 OpenAI 协议类 API，接入任意模型
- 每次会话后自动积累经验，通过 `/evolve` 快环和 `/fine-tune` 慢环持续自我进化

## 与竞品的差异

| 工具 | 进化方式 | 特点 |
|:--|:--|:--|
| Claude Code / Codex | 无内建进化，session 间无记忆 | 强但静态 |
| Cursor | 无内建进化 | 依赖手动 prompt |
| **MOMO CODE** | **双速并行：秒级经验注入 + 周期性能力跃迁** | **默认完全免费** |

安装：`curl -fsSL https://momozi.cc/install | bash`
