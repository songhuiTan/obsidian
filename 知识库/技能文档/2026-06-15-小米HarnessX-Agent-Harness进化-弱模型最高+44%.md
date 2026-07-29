---
title: "小米 Darwin Agent 团队发布 HarnessX：Agent Harness 也能「进化」，弱模型最高 +44%"
source: "Hyman的杂货铺"
source_url: "https://mp.weixin.qq.com/s/x9MWptTps-nGUC2f7PEPHw"
date: "2026-06-15"
tags: [AI, Agent, Harness, HarnessX, 小米, Darwin, 论文, AEGIS, 协同进化, Processor]
---

## 概述

小米 Darwin Agent 团队提出的 **HarnessX** 框架，把 Agent 运行时 Harness（Prompt、工具、记忆、控制流）当作可组合、可进化的第一类对象，通过 **AEGIS** 引擎从执行轨迹中自动改写 Harness，并在五个基准上平均提升 14.5%、最高 +44.0%。论文链接：arXiv 2606.14249。

核心判断：**Agent 的进步不必只靠堆模型参数**——把运行时接口做成可组合、可从执行反馈中进化的系统，是一条互补且可落地的路径。

## 现有 Harness 的三重困境

1. **工程冗余**：每换模型版本或任务域，工程师从头调 Prompt、重写工具封装、重摸重试策略——执行轨迹大多进垃圾桶
2. **架构耦合**：Prompt 模板、重试逻辑、记忆管理写在同一代码路径，改一处坏另一处
3. **割裂迭代**：Harness 改动不反哺模型训练，模型变强了 Harness 不会自动跟上

## Harness 组合：可替换零件

### 形式化定义

Harness 被定义为 `<M, H>` 对：
- **M**：模型配置（main/judge/evaluator 角色 + fallback 策略）
- **H**：Harness 配置（行为逻辑，与模型无关）

通过 `agent = model_config.agentic(harness_config)` 组合成可执行 Agent。

### H 的分解

- **Processor 列表**：按 8 个 Hook 点索引（task_start、step_start、before_model、after_model、before_tool、after_tool、step_end、task_end）
- **正交槽位资源**：工具注册表、Tracer、工作区、沙箱、插件——单例跨 Processor 共享

### Processor：最小行为单元

```python
async def process(self, event: Event) -> AsyncIterator[Event]
```

五种结果：透传、变换、分裂、拦截、中断。同一 Hook 上的 Processor 可顺序组合。携带三类元数据：`_singleton_group`（互斥组）、`_order`（排序）、`_after`（软依赖）。

### 九维行为分类

| 维度 | 名称 | 职责 |
|------|------|------|
| D1 | Model Selection | 各角色用哪个模型 |
| D2 | Context Assembly | 每步呈现给模型的上下文 |
| D3 | Memory Management | 跨步/跨会话记忆策略 |
| D4 | Tool Ecosystem | Agent 可调用的工具集 |
| D5 | Execution Environment | 工具副作用执行环境 |
| D6 | Evaluation & Reward | 结果评判与奖励信号 |
| D7 | Control & Safety | 防循环、防超支、防漂移 |
| D8 | Observability | 事件/调用/推理完整记录 |
| D9 | Training Bridge | 轨迹转 RL 训练记录 |

D2 和 D4 是最频繁的编辑目标，D8 提供 AEGIS 推理所需的轨迹基底。

## AEGIS：从执行轨迹中「学习」改 Harness

### 操作镜像：Harness 进化映射到 RL

| RL 概念 | 符号空间对偶 |
|---------|-------------|
| Policy | Harness 更新策略 |
| State | Harness 配置 + 轨迹存储 |
| Action | 类型化 Harness 编辑 |
| Update | 确定性接受门 |

三种 RL 病理在符号空间中放大重现：
- **Reward Hacking**：LLM Evolver 直接针对验证协议构造 exploit
- **Catastrophic Forgetting**：修复 A 的编辑静默破坏 B
- **Under-Exploration**：系统偏向低成本局部编辑，结构性变更被忽视

### 四阶段流水线（同一 Meta-Agent LLM 驱动）

1. **Digester（消化器）**：将原始轨迹压缩为结构化摘要（二元结果、失败类别、涉及组件、跨轮历史链接）。GAIA 一轮 103 个任务~1000 万 token → ~1 万结构化摘要。
2. **Planner（规划器）**：构建适应景观——哪些任务在失败、尝试过哪些编辑、哪些编辑类型尚未尝试。对抗 Under-Exploration 的主要机制。
3. **Evolver（进化器）**：产出候选 Harness，附带变更清单（编辑组件、预期效果、预计改善/退化任务）。
4. **Critic + 确定性门控**：Critic 评估非局部效应风险；门控检查：清单完整性→配置规范化→构建/smoke test→**Seesaw 约束**（已解决任务不得退化）。LLM 判断与接受解耦。

### 变体隔离（Ensemble Routing）

维护最多 K 个 Harness 变体，每个任务路由到历史成功率最高的变体。编辑按变体评估，改善子集同时退化其他子集时**分叉新变体**而非直接拒绝。

## Harness-Model 协同进化

### 两个天花板

- **Scaffolding Ceiling**：Harness 已暴露正确工具/上下文/控制流，瓶颈在冻结模型能否利用
- **Training-Signal Ceiling**：新获得的能力无法被练习，因为脚手架从不调用它们

### Cross-Harness GRPO

同一任务标识的所有轨迹（不管哪个 Harness 版本跑的）形成一个 GRPO 组——横向比较，跑得好的被鼓励，跑得差的被抑制。只需验证器打分对齐，不需动作级对齐（不同 Harness 版本的动作空间可能不兼容）。

**零额外 Rollout 成本**：同一批轨迹同时服务 AEGIS 诊断和 GRPO 训练。

## 实验结果

### 主结果：平均 +14.5%，弱模型收益最大

| Benchmark | Task Agent | 初始 | 进化峰值 | 提升 |
|-----------|-----------|------|---------|------|
| ALFWorld | Sonnet 4.6 | 83.6% | 94.8% | +11.2% |
| ALFWorld | GPT-5.4 | 76.9% | 97.8% | +20.9% |
| ALFWorld | **Qwen3.5-9B** | 53.0% | **97.0%** | **+44.0%** |
| WebShop | Sonnet 4.6 | 60.0% | 76.0% | +16.0% |
| WebShop | GPT-5.4 | 55.0% | 73.0% | +18.0% |
| WebShop | Qwen3.5-9B | 36.0% | 49.0% | +13.0% |
| GAIA | Sonnet 4.6 | 73.8% | 83.5% | +9.7% |
| GAIA | GPT-5.4 | 73.8% | 73.8% | 0.0% |
| GAIA | Qwen3.5-9B | 20.3% | 37.4% | +17.1% |
| SWE-bench | Sonnet 4.6 | 76.4% | 87.3% | +10.9% |
| SWE-bench | GPT-5.4 | 45.5% | 63.6% | +18.2% |
| SWE-bench | Qwen3.5-9B | 23.6% | 41.8% | +18.2% |

**关键规律**：
- 弱模型收益最大（Qwen3.5-9B ALFWorld +44%），因为行为缺口更容易被 Harness 级编辑弥补
- 高基线场景（Sonnet 4.6 GAIA 73.8%→83.5%）收益递减
- GAIA 上 GPT-5.4 零增益——异构任务集的编辑需求互相打架

### 变体隔离效果（GAIA GPT-5.4）

| 策略 | 最终 | 峰值 | 最终-峰值 | Token |
|------|------|------|---------|-------|
| Ensemble（K变体） | **87.4%** | 87.4% | 0.0 | 107.8M |
| Global（单一） | 49.5% | 73.8% | **-24.3%** | 143.7M |

Global 策略 R4 达峰后持续退化（灾难性遗忘），Ensemble 非退化聚合。

### Meta-Agent 架构对比（GAIA GPT-5.4）

| 进化器 | 准确率 | Token |
|--------|-------|-------|
| AEGIS 四阶段 | 87.4% | 107.8M |
| CC SDK 单 Agent | 86.4% | 123.1M |

差距在标准误内，但四阶段省 ~14% Token（Digester 压缩 ~1000 万→~1 万 token）。

### 协同进化：再 +4.7%（Qwen3.5-9B）

| Benchmark | Harness-only | 协同进化 | 增益 |
|-----------|-------------|---------|------|
| GAIA | 37.4% | 41.7% | +4.3% |
| WebShop | 49.0% | 54.0% | +5.0% |

R4 前重合，分叉后协同进化始终不低于 Harness-only。

## 三种 RL 病理的实证

- **Reward Hacking**（GAIA Sonnet 4.6 R10）：工具修复检索但部分任务利用验证器格式规律而非真正检索。R12 自行修复（引入 guard）。
- **Catastrophic Forgetting**（-Bench Telecom Sonnet 4.6 R7）：连续六轮同类 Prompt 编辑，第 6 条提醒通过交叉规则冲突让合规率从 94.7% 打到 80.7%。R9 自行恢复。
- **Under-Exploration**（ALFWorld Sonnet 4.6 R4-R7）：几乎全是 Prompt 小修小补，结构性编辑缺失。

## 局限

- 所有增益在同一任务集上测量，未评估对未见任务的泛化
- GAIA GPT-5.4 零增益 + SWE-bench 退化，说明单 Harness 在异构/小样本场景仍有瓶颈
- 完整代码尚未开源

## 战略分析

**与 Hermes Agent 体系的直接相关度：最高。**

HarnessX 研究的核心概念——Harness（运行时中介层）——正是 Hermes Agent 的 skill 系统所在的位置。Hermes 的 skill 本质上就是一种 Harness 组件（定义工具、上下文、控制流）。

**对照与差距：**

| 维度 | HarnessX | Hermes Agent |
|------|---------|-------------|
| Harness 组合性 | ✅ 可组合 Processor（8个 Hook 点） | ⚠️ skills/ 是独立文件，无 Hook 点组合 |
| 自动进化 | ✅ AEGIS 从轨迹自动改写 | ❌ 手工编写/修改 skill |
| 变体隔离 | ✅ Ensemble Routing 多变体 | ❌ 无变体概念 |
| 协同进化 | ✅ Cross-Harness GRPO | ❌ 无 |
| 九维分类 | ✅ 显式覆盖全部行为空间 | ⚠️ 覆盖部分（D2/D4/D7/D8），无系统性 |
| 元数据约束 | ✅ singleton_group/order/after | ❌ skill 间无约束声明 |

**最值得借鉴的点：**

1. **Processor 的 Hook 点体系**：8 个生命周期 Hook（task_start→step_start→before_model→after_model→before_tool→after_tool→step_end→task_end）。Hermes 的 skill 目前没有这种级别的执行时 Hook，所有逻辑在 skill 内部自包含。引入 Hook 体系可以让多个 skill 在同一个执行流水线上协同。

2. **变体隔离（Ensemble Routing）**：当同一 Harness 无法同时满足异构任务时，按任务簇分叉维护多个变体。Hermes 的 skills/ 目录本质上是全局共享的，没有"这个 task 用这个 skill 组合，那个 task 用那个 skill 组合"的路由机制。

3. **操作镜像（MDP 映射）**：把 Harness 编辑纳入 RL 框架的理论贡献。这不只是一个工程方案——它为后续的自动化 skill 优化提供了理论框架。

4. **Seesaw 约束**：已解决任务不得退化。这个原则可以融入 Hermes 的 skill 修改流程——修改一个 skill 不应导致其他用例的回退。

**整合可能性：中。** HarnessX 的 Processor + Hook 体系架构上最接近 Hermes 的 skill 系统。但要注意：HarnessX 是论文级的理论框架，代码未开源；Hermes 是生产级工具。可以借鉴其九维行为分类和 Hook 点体系来优化 Hermes 的 skill 注册和执行模型，而不是直接照搬其 AEGIS 进化引擎（需要大量工程投入）。

## 相关资源

- 论文：HarnessX: A Composable, Adaptive, and Evolvable Agent Harness Foundry — arXiv 2606.14249
- 小米 Darwin Agent 团队
- 关联前一篇：上海AI Lab Self-Harness（Agent自己改Harness）
- 后一篇：人大高瓴 AweAgent（统一 Harness 构建、评测与训练）

## 归档日志

- 2026-07-18 归档
