# 会自进化的 AI Agent：MLEvolve 如何自己发现算法

> 来源：论文收割机（微信公众号）
> 日期：2026-06-07
> 链接：https://mp.weixin.qq.com/s/FSr56CArua7c5eLKiItfxg
> 论文：https://arxiv.org/abs/2606.06473
> 代码：https://github.com/InternScience/MLEvolve

---

MLEvolve: A Self-Evolving Framework for Automated Machine Learning Algorithm Discovery。将 LLM Agent 从"单次代码生成器"推进成带搜索图、记忆和多种代码编辑模式的自动 ML 工程系统。

## 核心结果

MLE-Bench 75 个任务上，12 小时预算达到 **65.3% average medal rate**，官方 README 标注为 MLE-Bench 榜单第一。

## 三大核心设计

### 1. Progressive Monte Carlo Graph Search（渐进式 MC 图搜索）

不是树而是**图**——分支之间可以共享信息，避免重复踩坑。

`SearchNode` 不只保存代码，还保存 plan、metric、bug 状态、stage、branch_id、children、reward 等搜索元数据。

搜索策略软切换：`get_exploration_weight()` 前半程多试方向（探索），中后程逐渐押注表现好的候选（利用）。

```
前半程（0-50%）：全面探索
中段（50-70%）：逐渐衰减探索权重
后半程（70%+）：只押注高分方案
```

### 2. Retrospective Memory（回顾式记忆）

**不是聊天记录，而是实验经验。**

两层结构：
- **cold-start domain knowledge base**：初始阶段为任务类型推荐已有经验
- **dynamic global memory**：保存搜索过程中每个有效节点的 plan、代码摘要、metric、成功/失败标签

核心机制：Planner 生成初始 plan → 检索相似成功记录和失败记录（各 Top-2）→ 用于 refine plan。成功/失败标签可过滤，历史实验变成可检索、可过滤、可用于规划的经验库。

### 3. Hierarchical Planning with Adaptive Code Generation

"想做什么"和"怎么改代码"拆开。

三种生成模式：
- **base_coder**：一步式 plan + code generation
- **stepwise_coder**：多 Agent 分步生成（数据→模型→训练等模块）
- **diff_coder**：基于 diff 的代码生成（SEARCH/REPLACE 式），适合长周期演化

## 停滞检测与策略切换

关键判断：分支是否停滞（stagnant）。

```
if 分支停滞:
    if evolution 和 fusion 都可用 → 按概率选 evolution/fusion
    elif fusion 可用 → fusion（从别的分支拿成功经验）
    elif evolution 可用 → evolution（沿当前分支历史轨迹做进化）
else:
    improve（普通优化）
```

每次改进分为三层：
- **Tier 1**：训练细节优化（学习率、batch size）
- **Tier 2**：组件和表示变化
- **Tier 3**：系统级范式转移（模型类型、目标建模方式）

连续不涨时，要求模型提出 Tier 2 或 Tier 3 的变化。

## 从业者启发

1. **别把 memory 做成聊天记录**——真正有用的记忆是实验记录（plan、代码摘要、metric、标签）
2. **搜索结构比 prompt 更重要**——系统要定义何时探索、何时利用、何时跨分支融合
3. **代码生成要分模式**——初始方案全文生成，成熟方案 diff patch，复杂方案 stepwise
4. **停滞检测显式化**——分数不涨时触发更大幅度的策略变化，而非继续随机微调

## 系统配置

```
agent:
  steps: 500
  time_limit: 43200  # 12 小时
  use_diff_mode: True
  use_stepwise_generation: True
  use_evolution: True
  use_fusion: True
  use_global_memory: True
```

资源：每个任务最多 500 expansion steps、12h、21 vCPU、234GB RAM、单张 H200 GPU。
