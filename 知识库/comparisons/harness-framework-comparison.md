---
title: "Harness 框架对比 — 主流 Agent Harness 实现横评"
created: 2026-07-21
updated: 2026-07-21
type: comparison
tags: [comparison, harness, agent-framework, orchestration, skill-system]
sources:
  - 技能文档/主流Agent-Harness实现对比-SubAgent与MultiAgent.md
  - 技能文档/Trellis-团队级Agent-Harness框架-拆解vs-Superpowers.md
  - 技能文档/OpenSpace-Skill进化-HKUDS-6600Star.md
  - 技能文档/2026-07-15-拆完WorkBuddy-我看到了生产级Agent的完整形态.md
  - 技能文档/2026-06-15-小米HarnessX-Agent-Harness进化-弱模型最高+44%.md
  - 技能文档/2026-07-06-从Vibe-Coding到Harness-腾讯大仓AI工程化实战.md
  - 技能文档/2026-07-03-Harness-Engineering-AI-Agent从Demo走向生产级可靠性.md
confidence: medium
---

# Harness 框架对比

## 主流 Harness 实现横向对比

| 对比维度 | Trellis | OpenSpace | WorkBuddy | HarnessX（小米） | TAB（腾讯） |
|----------|---------|-----------|-----------|-----------------|-------------|
| 核心定位 | 团队级 Harness + LLM Wiki | 自进化 Skill 引擎 | 生产级 Agent 方法论 | Harness 进化框架 | 大仓 AI 工程化实践 |
| 架构层次 | 三层：执行骨架/LLM Wiki/团队协作 | Skill 进化引擎 | 六阶段演进 | AEGIS 四阶段进化 | 六层资产 |
| 核心创新 | 文件即记忆，Spec git 版本化 | Skill FIX/DERIVED/CAPTURED 进化 | 场景价值公式 + 六阶段 | 8 Hook 点 Processor 体系 | 13 阶段接力赛 |
| 状态管理 | 跨会话状态机 | Skill 谱系记录 | Task/Step/Status/Artifact | 变体隔离 | 7 道门禁 |
| 团队协作 | Spec 库 git 版本化共享 | Skill 进化共享 | 方法论指导 | 论文/学术 | 腾讯内部大仓 |
| 跨工具适配 | 16 个平台 | 可接入 Claude Code/Codex 等 | 通用方法论 | 通用（弱模型友好） | 腾讯内部 |
| 评测/基准 | — | GDPVal (收入提升 4.2x) | 场景价值公式 | 5 基准平均 +14.5% | 内部门禁 |
| Star/影响力 | GitHub 12k Star | 6,660 Star | 方法论传播 | 论文引用 | 腾讯内部 |

## 各框架深入解析

### Trellis（三层架构）
GitHub 12k Star 的团队级 Harness 框架。三层设计：Agent Harness 执行骨架（管理 workflow 状态/hook/skill/子代理调度）、内置 LLM Wiki（Spec + Task + Journal 文件化）、团队协作层（git 版本化，16 平台适配）。trellis-meta 的五层防御体系（意图澄清→路线锁定→方案挑刺→任务契约→测试先行）为大型项目提供了完整方法论。参考 [[trellis]] 实体文档。

### OpenSpace（Skill 进化引擎）
HKUDS 出品，6,660 Star，MIT 协议。核心命题：Skill 不应是静态文件，而应在每次使用中自动进化。三种进化模式：FIX（修复失效 Skill）、DERIVED（派生细分 Skill）、CAPTURED（捕获可复用工作流为 Skill）。三个进化触发器：执行后分析、工具退化检测、周期性指标检查。Benchmark: 50 个任务收入提升 4.2 倍，Token 用量降至 45.9%。参考 [[openspace]] 实体文档。

### WorkBuddy（六阶段方法论）
叶小钗系统阐述的生产级 Agent 方法论。核心主张"场景 > 框架"。六阶段演进路径：能聊→能用工具→能做任务→能跑长任务→能被治理→能沉淀能力。工程五大系统：大脑系统（Model + Loop）、行动系统（Tools + Runtime）、工作台系统（State + Context + Memory）、经验系统（Prompt + Skill + Workflow）、控制系统（HITL + Guardrails + Eval）。参考 [[workbuddy]] 实体文档。

### HarnessX（小米，AEGIS 四阶段进化）
小米 Darwin 团队的论文工作。核心思想：把 Harness（Prompt/工具/记忆/控制流）作为可组合、可进化的第一类对象。AEGIS 四阶段进化引擎 + 8 个 Hook 点 Processor 体系 + 九维行为分类 + 变体隔离。81 个配置跨 5 基准平均 +14.5%，弱模型最高 +44%。证明：Harness 质量比模型参数更重要。

### TAB（腾讯，六层资产 + 13 阶段接力赛）
腾讯大仓 AI 工程化实战。六层资产：Rule / Skill / SubAgent / Workflow / Scripts / MCP。13 阶段接力赛流程，4 Agent 职责契约 + 7 道门禁（基线对比反作弊）+ 5 个人工关卡（半自动）+ 5 个 MCP 打通交付闭环。代表了互联网大厂将 Harness 工程落地的最高水平。

## 关键维度对比

| 维度 | Trellis | OpenSpace | WorkBuddy | HarnessX | TAB |
|------|---------|-----------|-----------|----------|-----|
| 团队协作 | ★★★★★ | ★★★ | ★★★ | ★★ | ★★★★ |
| Skill 进化 | ★★★ | ★★★★★ | ★★★ | ★★★★ | ★★★ |
| 生产级方法论 | ★★★★ | ★★★ | ★★★★★ | ★★★ | ★★★★ |
| 弱模型兼容 | ★★ | ★★ | ★★★ | ★★★★★ | ★★★ |
| 技术深度 | ★★★ | ★★★★ | ★★★ | ★★★★★ | ★★★★ |

## 选型建议

- **小团队需要协作基础设施** → [[trellis]]（开箱即用的团队 Harness）
- **需要 Skill 自动进化能力** → [[openspace]]（嵌入现有 Agent 平台）
- **希望建立生产级 Agent 方法论** → [[workbuddy]]（六阶段指导落地）
- **弱模型优化/学术研究** → HarnessX（Hook 进化体系 + 强模型兼容性）
- **大型企业/大仓落地** → TAB 腾讯方案（六层资产 + 严格门禁）

## 综合趋势

Harness 工程正在从单一框架走向分层互补体系。Trellis 管团队协作，OpenSpace 管 Skill 进化，WorkBuddy 管生产方法论，HarnessX 管框架进化，TAB 管企业落地——它们在不同维度解决 Agent 工程化问题。未来趋势是这些能力在同一个 Harness 中融合，形成"执行骨架 + 知识库 + 进化引擎 + 治理门禁"的完整体系。
