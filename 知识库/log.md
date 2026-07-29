# Wiki Log

> Chronological record of all wiki actions. Append-only.
> Format: `## [YYYY-MM-DD] action | subject`
> Actions: schema-update, lint, index-update, nav-update, ingest, query, create, archive

## [2026-07-03] schema-create | Wiki 初始化
- Architecture: Option B (集成到现有 Obsidian 知识库 Vault)
- SCHEMA.md created with domain, conventions, tag taxonomy, and 红绿灯原则
- 用户内容目录 (技能文档/ 教程指南/ 复盘日记/ 等) 标记为 Layer 1 不可变内容
- 后续任务：更新 README 计数，补全快速导航缺失文章

## [2026-07-03] nav-update | 补全快速导航缺失文章 (43篇)
- README: 技能文档 133→159篇, 教程指南 4→5篇, 合计 133→169篇
- 快速导航: Agent框架 +16, Claude Code +4, RAG +7, 知识管理 +1, 工具 +9, 企业 +3, OpenClaw +1, 教程 +2
- 创建 SCHEMA.md, index.md, log.md (llm-wiki Option B 架构)
- WIKI_PATH 已设为 /Users/mac/Documents/workplace/obsidian/知识库

## [2026-07-03] housekeeping | 合并 assets 重复目录
- 删除 assets/2026-05-07-用Agent评测思路管理AICoding（与 2026-05-07-Agent评测思路管理AI Coding 内容完全相同且未被引用）
| - WIKI_PATH 写入 ~/.hermes/.env

## [2026-07-21] wiki-build | 完整构建 Layer 2 Wiki（32 页）
- 从 269 篇技能文档中提炼并创建 32 个 wiki 页面
- Entities（14 页）：LangGraph / DeepAgents / Hermes Agent / Claude Code / Codex / AgentScope / Langfuse / OpenClaw / Trellis / OpenSpace / PilotDeck / WorkBuddy / OpenDev / CoreCoder
- Concepts（13 页）：Harness Engineering / Loop Engineering / Skill 体系 / Agent 评测体系 / Agent 治理 / 多 Agent 编排 / Agent 记忆系统 / RAG 架构谱系 / Agent 可观测性 / Context Engineering / MCP / HITL / SubAgent 模式
- Comparisons（5 页）：三大框架对比 / Claude Code vs Codex / AgentScope vs LangGraph / RAG 架构演进对比 / Harness 框架对比
- 已更新 index.md 反映所有新页面
- 知识来源：269 篇技能文档归档（Layer 1）
|