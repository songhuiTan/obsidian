# Wiki Index

> LLM Wiki 内容索引（Layer 2）。Agent 管理的实体/概念/对比/查询页面。
> 用户自有内容请查阅 [[README.md]] 或 [[快速导航.md]]
> 最后更新：2026-07-21 | 总页数：32

## 知识库来源映射

以下是 Vault 中已有的用户内容（Layer 1），wiki 页通过 `sources:` frontmatter 引用：

| 目录 | 类型 | 路径前缀 |
|------|------|---------|
| 归档文章 | 技能/工具/框架分析（269 篇） | `技能文档/` |
| 教程指南 | 完整教程 | `教程指南/` |
| 复盘日记 | 学习复盘 | `复盘日记/` |
| 平台经验 | InStreet 经验 | `InStreet经验/` |
| 系统配置 | OpenClaw 系统 | `OpenClaw系统/` |
| 技能学习 | ClawBot 技能 | `ClawBot技能学习/` |
| 索引 | 分类索引 | `索引目录/` |
| 资源 | 图片/备份 | `assets/` |

## Entities（14 页）

| 页面 | 摘要 |
|------|------|
| [[entities/agentscope\|AgentScope]] | 阿里达摩院生产级多智能体框架，事件驱动+ReAct/17+Provider/MCP+A2A/Actor分布式 |
| [[entities/claude-code\|Claude Code]] | Anthropic 出品的终端 Agent 编程工具，Loop 工程+Skill 系统+多 Agent 编排 |
| [[entities/codex\|CodeX]] | AI 编程终端 Agent，无限画布+Workflow 继承，对标 Claude Code 的开源生态 |
| [[entities/corecoder\|CoreCoder]] | 1,714 行 Python 复刻 Claude Code 核心的教育级开源项目 |
| [[entities/deep-agents\|DeepAgents]] | LangChain 深度自主 Agent 框架，虚拟文件系统+17 内置中间件+SubAgent |
| [[entities/hermes-agent\|Hermes Agent]] | 开源 Agent 框架，OPC 多角色架构（协调/研究员/写作者/构建者） |
| [[entities/langfuse\|Langfuse]] | 开源 LLM 工程平台，可观察性+Prompt 管理+评估，ClickHouse 后端 |
| [[entities/langgraph\|LangGraph]] | LangChain 有状态图编排框架，显式控制流+DAG/图执行引擎 |
| [[entities/openclaw\|OpenClaw]] | 清华大学开源 Agent 平台/社区，含教育版/技能市场/DeerFlow |
| [[entities/opendev\|OpenDev]] | Rust 开源终端 Agent，81 页论文全架构拆解，五层安全+ReAct Harness |
| [[entities/openspace\|OpenSpace]] | HKUDS 6.6K Star Skill 进化框架，FIX/DERIVED/CAPTURED 生命周期 |
| [[entities/pilotdeck\|PilotDeck]] | 清华+面壁智能开源多 Agent OS，WorkSpace 隔离+Dream 模式+智能路由 |
| [[entities/trellis\|Trellis]] | 12K Star 团队级 Agent Harness，三层架构（执行骨架/LLM Wiki/团队协作） |
| [[entities/workbuddy\|WorkBuddy]] | 生产级 Agent 方法论，六阶段演进路径+双视角架构 |

## Concepts（13 页）

| 页面 | 摘要 |
|------|------|
| [[concepts/agent-evaluation\|Agent 评测体系]] | 从 Demo 到生产的 Agent 质量评估：五维度/三层 Grader/回放评测 |
| [[concepts/agent-governance\|Agent 治理]] | 生产环境 Agent 管控：Hook 切面治理/HITL/Guardrails/权限引擎 |
| [[concepts/agent-observability\|Agent 可观测性]] | Agent 运行轨迹追踪：Trace/Span/Generation 三层+OTel 采集 |
| [[concepts/context-engineering\|Context Engineering]] | AI 编码上下文的工程化优化：CEK/Headroom/SkillWeaver |
| [[concepts/harness-engineering\|Harness Engineering]] | Agent 生产级可靠性系统工程：六组件/Rule/Skill/SubAgent/Workflow |
| [[concepts/hitl\|HITL]] | Agent 执行中的人工介入机制：Hook 链护栏/配置驱动门禁 |
| [[concepts/loop-engineering\|Loop Engineering]] | Agent 自动化循环方法论：Observe→Plan→Act→Verify→Reflect |
| [[concepts/mcp\|MCP]] | Model Context Protocol：模型上下文协议/工具接入标准接口 |
| [[concepts/memory-systems\|Agent 记忆系统]] | 跨会话上下文保持：L0-L3 分层记忆/Checkpoint 持久化 |
| [[concepts/multi-agent-orchestration\|多 Agent 编排]] | 多 Agent 协同编排模式：并行分发/SubAgent/Leader-Worker |
| [[concepts/rag-architecture\|RAG 架构谱系]] | Naive→Agentic RAG 八种演进架构+前沿变体 |
| [[concepts/skill-system\|Skill 体系]] | Agent 能力可复用工程单元：从 Prompt→Skill 工程演进 |
| [[concepts/subagent-pattern\|SubAgent 模式]] | 复杂任务分解给子 Agent 的编排模式：上下文隔离+专门化 |

## Comparisons（5 页）

| 页面 | 摘要 |
|------|------|
| [[comparisons/langgraph-vs-deepagents-vs-hermes\|三大框架对比]] | LangGraph vs DeepAgents vs Hermes：图状态机/深度自主/环境构建 |
| [[comparisons/claude-code-vs-codex\|Claude Code vs Codex]] | 两大 AI 编程终端 Agent：功能/工作流/生态全面对比 |
| [[comparisons/agentscope-vs-langgraph\|AgentScope vs LangGraph]] | 企业级多智能体 vs 图编排：事件驱动/状态机选型 |
| [[comparisons/rag-architecture-comparison\|RAG 架构演进对比]] | 八种 RAG 架构选型决策：复杂度/幻觉控制/场景匹配 |
| [[comparisons/harness-framework-comparison\|Harness 框架对比]] | Trellis/OpenSpace/WorkBuddy/HarnessX/腾讯 TAB 横向对比 |

## Queries

<!-- Wiki 查询结果将列在这里 -->
