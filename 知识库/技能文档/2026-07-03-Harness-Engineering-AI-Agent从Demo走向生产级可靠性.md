---
title: "Harness Engineering：让 AI Agent 从 Demo 走向生产级可靠性的关键"
author: "AI研究生 / AI大模型观察站"
date: "2026-07-03"
source: "https://mp.weixin.qq.com/s/jtuykooSPtcH9KLWmV96Dw"
tags: [Harness, Agent架构, 生产级可靠性, 工程化, 状态管理, 评估, 编排]
---

# Harness Engineering：让 AI Agent 从 Demo 走向生产级可靠性的关键

> AI Agent 失败往往不是模型不够强，而是缺少 Harness。本文解析生产级 Agent 所需的约束、编排、状态、评估与恢复体系。

## 核心认知：Agent = Model + Harness

几乎所有让 Agent **真正能在生产环境中工作**的东西——除了 foundation model API call 本身之外——都是 Harness。

**类比：派一名初级员工去主持关键客户会议**
- Prompting = 告诉他议程
- Context = 把资料包交给他
- **Harness = 其他所有东西**：checklist、中途 check-in、录制的 transcript、偏离脚本时的纠偏机制、报告验收标准

**突破性结果**：通过重构 task decomposition、state management、critical step validation 和 failure recovery，作者用同一个底层模型和同样的 prompts，把任务成功率从 68% 推到了 95% 以上。

## AI Engineering 的三个阶段

| 阶段 | 核心关注 | 天花板 |
|------|---------|--------|
| 1. Prompt Engineering | "模型理解我了吗？" | 无法弥补事实 grounding 的缺失 |
| 2. Context Engineering | "模型掌握事实了吗？" | 无法解决 execution drift（执行漂移） |
| 3. **Harness Engineering** | "模型能持续采取正确行动吗？" | — |

> 前两个阶段帮助模型更好地思考，Harness Engineering 确保它 **可靠地行动**。

## 成熟 Harness 的六个架构层

### 第 1 层：Information Boundaries（认知范围）
- 多余的数据不会让模型更聪明，只会让它失去焦点
- 必须明确定义并分类模型看到的内容：角色、目标、成功标准、不同信息类型之间的结构化分离

### 第 2 层：Tool System（执行能力）
- 给模型太多 tools 会分散注意力、导致 hallucinate 不存在的参数
- Harness 必须控制 **何时使用 tools**，而不仅仅是哪些 tools 可用
- **不可协商**：永远不要把原始 tool outputs 直接回传给 LLM——须过滤、解析和总结

### 第 3 层：Execution Orchestration（规划与路由）
- LLM 失败往往不是因为缺少单项技能，而是无法串联
- Harness 铺设严格轨道：
  > Understand Goal → Assess Information → Fetch Missing Info → Analyze → Generate → Verify → Output
- **项目管理责任从概率模型转移到确定性系统**

### 第 4 层：Memory and State（连续性）
三种严格隔离的 memory 类型：
1. **Current Task State** — 当前步骤、待处理项、已确认项
2. **Conversational Intermediate Results** — 本次 session 已有结论
3. **Long-Term Memory / User Profiles** — 跨 session 的全局偏好

> ⚠️ 常见陷阱：把 task state 和 conversational history 混为一谈，导致 context window 无限增长。

### 第 5 层：Evaluation and Observability（自我感知）
- 评估自己工作的 agent 有深刻的乐观偏差
- 需要独立自动化验证机制：输出验证、集成测试环境、细致 logging、metrics tracking、error attribution

> 系统必须持续向自己证明它的行动是正确的，而不是仅仅假设它们正确。

### 第 6 层：Constraints, Validation, and Recovery（韧性）
- 在生产环境中，失败是默认状态
- 三件必需品：
  - **Constraints**：硬编码的禁止规则
  - **Validation**：输出前/后的 gating checks
  - **Recovery**：Retry logic、fallback paths、回滚能力

## Context Anxiety（上下文焦虑）

Anthropic 研究发现：当 context window 接近限制时，模型开始丢失细粒度细节、忘记核心目标、表现得急于完成任务——跳过验证、hallucinate 结论。

**有效方案：Context Reflect** — 当 context 太大时，取出压缩后的 summary，交给一个**完全新的 agent instance**（干净的 context）。原理与处理 memory leak 相同：重启进程，而不是疯狂 GC。

与之配套的 **Progressive Disclosure**：一开始只给模型最少的 tool stubs，当它表达出使用某个工具的意图时，再动态注入详细文档和参数 schemas。

## 将生成与评估分离

Anthropic 构建真正自主 Agent 的关键架构——**严格三方拆分**：

1. **Planner** — 将模糊需求转化为严谨的工程规格
2. **Generator** — 接收规格并逐步执行
3. **Evaluator** — 完全独立的 QA 实体，与 Generator 功能解耦

Evaluator 不只是读代码，而是与**渲染后的真实输出**交互（点击界面、检查布局、验证交互状态）。

> "Done" 不再意味着"我完成了文本生成"。它意味着"我运行了代码，review 了 logs，发现了一个 bug，修复了它，并在 sandbox 中验证了 deployment。"

## 总结

> Foundation model 的智能定义了它在 benchmark leaderboard 上的理论上限。**Harness Engineering 的鲁棒性决定了这种智能是否真的能在混乱的真实世界中生存、恢复并交付价值。**

70% 成功率与 95% 成功率之间的差距——demo 与 product 之间的差距——完全存在于 Harness 之中。
