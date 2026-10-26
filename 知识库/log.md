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
## [2026-09-23] archive | FDE案例库-127条企业AI落地案例的可检索证据库
- 来源: GitHub guminghao-avi/FDE-case-library (15⭐, MIT, 2026-09-19 开源, v0.2.0)
- 归档方式: GitHub REST API + raw 文件读取 (未 clone), 全量分析 cases.json 532KB/127条
- 文档: 技能文档/2026-09-23-FDE案例库-127条企业AI落地案例的可检索证据库.md
- 图片: assets/2026-09-23-FDE案例库/img_001.png (首页截图 1440x1000)
- 导航: 快速导航 企业级AI&AIOps +1, 总数 351→352; README 技能文档 334→335
- 核心价值: analysis_boundary 字段级证据边界 + 六维详细度评分 + source_record 来源账本, 可借鉴进归档流程

## [2026-10-26] archive | PWA移动APP工作台提示词（飞书）
- 来源: 飞书 wiki https://zcnv48kqce5a.feishu.cn/wiki/WU84wbYdgi3uylkYvOVc6LKVn5d (docx token XzYRdfZDfobaAox9WUwcJT3kn0M)
- 归档方式: feishu-doc references/feishu-to-obsidian-archive.md — wiki get_node 取 obj_token → docx blocks API (7 块: 1 page + 4 text + 2 code)
- 文档: 技能文档/2026-10-26-PWA移动APP工作台提示词.md
- 图片: 无（原文无图无附件）
- 导航: 快速导航 知识管理与归档 +1, 总数 363→364; README 技能文档 362→363
- 核心价值: 两段式提示词结构（通用 PWA 8 条约束=可复用资产 / 岗位描述段=消耗品），附 PWA 技术规格核对清单
- 备注: 原文岗位示例为「小学教师工作台」；通用段可直接换岗位复用

## [2026-10-26] archive | StepCode+StepPage 三端工作台（抖音视频）
- 来源: 抖音 v.douyin.com/TQDrmzeCIgk → www.douyin.com/video/7694200099710192932 (李君陌, 2026-10-08 18:50, 20:58)
- 归档方式: 轻路径 — 页面 innerText 一次提取成功（章节要点8段含时间戳 + 元数据 + 评论），未做 ASR
- 官方口径补充: platform.stepfun.ai/docs/en/step-code/quickstart + .../reference/steppage + github.com/stepfun-ai/Step-Code (实测 1019⭐/68fork, MIT)
- 文档: 技能文档/2026-10-08-StepCode-StepPage-三端工作台-新手手搓手机可用工作台.md
- 图片: assets/2026-10-08-StepCode-StepPage-三端工作台/cover.jpg (页面 lark:url:video_cover_image_url, 440x330)
- 导航: 快速导航 视频归档 +1, 总数 364→365; README 技能文档 363→364
- 关键判断: 与 2026-10-26-PWA移动APP工作台提示词 构成上下游（生成层 vs 分发层）；视频口播"IP服务"实为 StepPage

## [2026-10-26] lint | 第1次 — 新 WIKI_PATH 首次健康检查（知识库/wiki 层）

- **口径变更说明：** 本周起 lint 扫描 `$WIKI_PATH` = `知识库/`，即 `知识库/{entities,concepts,comparisons,queries}` 32 页（14+13+5+0）。此前 17 次报告扫描的是 vault 根目录 `entities/ concepts/ queries/`（24 页）——那套结构已被 7/21 新建的 32 页体系取代，属于遗留结构。根目录页仍保留，其状态附在末尾一节。
- 扫描范围：32 个 Wiki 页（14 entities + 13 concepts + 5 comparisons + 0 queries）+ 知识库 Layer 1 共 422 篇（技能文档 364 / 教程指南 5 / 复盘日记 2 / InStreet 3 / ClawBot 1 / OpenClaw系统 1 / 索引 1 + 5 个 md 根文件）

### 🔴 CRITICAL（2 项）

- **断链 2 处**（有 2 个唯一目标不存在，链接写法本身只用纯标题名，未用 `entities/xxx` 路径，因此 64 处"疑似断链"实为 Obsidian 别名解析正常匹配）
  - `entities/openclaw.md` → `[[DeerFlow]]` — 无此页。修复：改为 `[[2026-04-15-DeerFlow-...]]` 指向技能文档，或新建 `entities/deerflow.md`
  - `entities/trellis.md` → `[[Superpowers]]` — 无此页。修复：改为指向 `技能文档/` 中 Superpowers 相关文章的 wikilink
- **红绿灯合规：`queries/` 目录为空（0 页）** — SCHEMA 的「queries 归档审批」黄灯条款当前无实体，无违规。但 7 篇 raw 级查询结论散落在 `知识库/` 根目录之外，未沉淀进 queries/。建议归档 2–3 条高价值查询（如 StepCode 三端工作台选型、Harness 框架对比结论）。
- **红灯合规：32 页全部 `confidence: medium`** — 无未经签字的 `high` 标记 ✅（对比旧结构 24 页全部 high 且连续 17 周未签字，新结构已自动规避该违规）。`contested` / `contradictions` 标记：0 页，无矛盾待裁决 ✅

### 🟡 HIGH（4 项）

- **过期内容 32/32 页（100%）** — 全部 `updated: 2026-07-21`（97 天），越过 90 天线。这是 7 月整批构建时的统一时间戳，从未更新过。修复：按 SCHEMA 更新规则，对本周有新增素材的主题页（见下条）逐页刷新 updated 并补内容。
- **Wiki 层与 Layer 1 严重脱节** — 32 页基于 7/21 的 269 篇素材，而知识库现有 364 篇（+95 篇，+35%）。10 月新增中已有现成摄入素材，全部未反映：
  - `2026-10-22-Raven一个入口管所有Agent-HarnessOfHarnesses` → `concepts/harness-engineering.md`
  - `2026-10-21-AOCI-CODE-代码地图-Git版本化仓库认知索引`、`2026-10-22-OneOrca-一个任务一个worktree 多Agent并行` → `concepts/multi-agent-orchestration.md`、`concepts/context-engineering.md`
  - `2026-10-21-context-mode上下文管理-MCP加hooks三层管道` → `concepts/context-engineering.md`、`concepts/mcp.md`
  - `2026-10-22-从LOOP到图探索-AutoResearch自进化系统架构演进` → `concepts/loop-engineering.md`
  - `2026-10-18-e2e-Agent驱动端到端测试`、`2026-10-20-AI写的代码谁来验-5个开源项目接管测试` → `concepts/agent-evaluation.md`
  - `2026-10-22-zg一个命令替代grep-阿里开源Agent检索层` → 新建 `entities/zg.md`（候选）
  - `2026-08-27-MindForge-基于LLM-Wiki的知识铸造平台` → `concepts/*wiki*` 或新建 entity（与本 wiki 自身模式直接相关）
- **索引内容过期** — `index.md` 的「知识库来源映射」表仍写 **269 篇**（实际 364 篇，差 95）；`Log.md` 头部 `Updated` 与 `总页数` 行未随 7 月构建后任何变更同步（最后更新 2026-07-21 / 总页数 32 与实物一致，但表内计数错误）。修复：更新来源映射表为 364 篇，并在「Queries」章节补注空状态说明。
- **标签越界 2 个** — `entities/langfuse.md`、其它 2 页使用的 `aiops`、`aigc`、`enterprise`、`codex`、`openclaw`、`hermes`、`mcp` 等中，**SCHEMA Tag Taxonomy 未列**：`aiops, aigc, enterprise, codex, hermes, openclaw, wiki, ppt, video, design, writing, tutorial, review, case-study` 为部分越界。实际使用的 22 个标签中，`agent-observability` 相关页用 `evaluation/governance/methodology/production/archive/knowledge-base/tool/memory/comparison` 均在册，越界集中在 `aigc, aiops, enterprise`。修复：SCHEMA 补入这 3 个（先加后用），或页面改用 `archive` / `production` / `integration`。

### 🟢 MEDIUM（3 项）

- **孤儿页 8 个（无任何入站链接）** — 全在 concepts/comparisons 区，且这 8 页恰好都是 7/21 新建、从未被 entities 侧引用：
  - `concepts/agent-observability.md`、`concepts/context-engineering.md`、`concepts/memory-systems.md`
  - `comparisons/agentscope-vs-langgraph.md`、`comparisons/claude-code-vs-codex.md`、`comparisons/harness-framework-comparison.md`、`comparisons/langgraph-vs-deepagents-vs-hermes.md`、`comparisons/rag-architecture-comparison.md`
  - 修复：在 `concepts/harness-engineering.md`、`concepts/skill-system.md` 加指向这 3 个 concepts 的链接；在 `entities/agentscope.md`、`claude-code.md`、`langgraph.md`、`rag-architecture.md` 加指向对应 comparisons 的链接。属绿灯区，可在下次 lint 自动补。
- **出站链接不足 2 条** — 本轮未发现低于 2 条的页面（32/32 达标 ✅）
- **量纲异常：根目录遗留结构** — vault 根 `entities/(16) concepts/(6) queries/(2)` 共 24 页自 9/14 起未再变动，其中断链 16 处 / 索引占位 24 个 / 全部 `confidence:high` 未签字，即旧第 1–17 次报告所指对象。**建议归档到 `_archive/legacy-wiki-root/` 并停止 lint**，否则两套 Wiki 并行会造成持续误读。

### 其他检查结果

- ✅ 大页面：0 页 >200 行（最长 `concepts/harness-engineering.md` 未超阈值）
- ✅ 索引完整性：`index.md` 32 条对应 32 个文件，无占位、无遗漏、无重复
- ✅ Frontmatter：32/32 完整（title/created/updated/type/tags/sources 齐备）
- ✅ 矛盾/争讼：`contested: true` 0 页，`contradictions:` 0 页
- ✅ `confidence: low`：0 页
- ✅ raw/ 源漂移：`知识库/` 无 raw/ 目录（7/21 构建未落地 raw 层）；vault 根 `raw/articles/` 7 个文件 sha256 全部匹配（口径为 `body.lstrip('\n')`，与存储一致）
- ⚠️ **Layer 1 计数一致性：** `README.md` 与 `快速导航.md` 均写 **365 篇**，实际为 364 篇（技能文档 364）。差 1 篇，属计数漂移。修复：两处同步为 364。
- ⚠️ **嵌套目录：** `知识库/知识库/技能文档/` 含 2 篇 10/23 REA 逆向分析（与 `知识库/技能文档/` 同名文章并存）。属 Layer 1 用户内容，只报告不处理；建议小谭确认后删除嵌套目录。
- ✅ Log 条目数 6（含本次），无需轮转
- 备注：`快速导航.md` 383 个 wikilink 中仅 1 个失效（`[[slug]]` 为模板占位符，非真实断链）

### 待决策（黄灯/红灯，需小谭确认）

1. **归档根目录遗留 Wiki 结构**（`entities/ concepts/ queries/` 共 24 页）→ 移入 `_archive/` 并停止双轨 lint。属绿灯区但涉及大量移动，建议确认。
2. **摄入 10 月新增素材**（8 篇候选，见 🟡 第 2 条）→ 可把 32 页过期问题与脱节问题一并解决。
3. **SCHEMA 标签补录**（`aigc / aiops / enterprise`）→ 黄灯区，需小谭确认后写入分类体系。

### 本次自动修复（绿灯区）
- ✅ `README.md` + `快速导航.md` 篇数计数 365 → **364**（与 `技能文档/` 实际 364 篇对齐）

## [2026-10-26] archive | 花生壳Drop + WorkBuddy PWA老师工作台（抖音视频）
- 来源: 抖音 v.douyin.com/GW5jD38s15s → www.douyin.com/video/7700446647565749513 (李君陌, 2026-10-25 11:45, 2:50)
- 归档方式: 轻路径 — innerText 一次提取（6 段章节要点 + 元数据 + 全部评论），未做 ASR
- 官方口径补充: hsk.oray.com/cases/cli（花生壳AI版/CLI）、fuyuan7.com 静态托管实测（index.html 约定/微信可开/免费）
- 文档: 技能文档/2026-10-25-花生壳Drop-PWA老师工作台-10分钟做出APP工作台.md
- 图片: assets/2026-10-25-花生壳Drop-PWA老师工作台-10分钟/cover.jpg (330x440)
- 导航: 快速导航 视频归档 +1, 总数 365→366; README 技能文档 364→365
- 关键发现: 作者评论区自述「花生壳是静态托管数据只能存本地，换云服务器才能挂数据库」
- 三方案定位成表: 花生壳Drop vs StepPage vs Cloudflare/Vercel；生成层通用，分发层才需选
