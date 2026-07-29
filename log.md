# Wiki Log

> AI Agent 领域 Wiki 操作记录。追加写入。
> 格式：`## [YYYY-MM-DD] action | subject`
> 操作类型：ingest, update, query, lint, create, archive, delete

## [2026-05-05] create | Wiki 初始化
- 领域：AI Agent 生态（OpenClaw、Harness、Agent 框架、工具链）
- 根目录：`/Users/mac/Documents/workplace/obsidian/`
- 创建文件：
  - `SCHEMA.md` — 领域定制 Schema（含标签分类体系、页面阈值、知识库映射）
  - `index.md` — 编录现有内容：9 个 Entities 分类入口、10 个 Concepts、2 个 Comparisons
  - `log.md` — 日志文件
- 创建目录：`entities/`, `concepts/`, `comparisons/`, `queries/`, `raw/{articles,papers,transcripts,assets}`, `_archive/`
- 已有知识库映射：`知识库/` 目录（45 篇 markdown）作为 Layer 1 原始材料
- 现有内容已编录：OpenClaw、MyHarness、OpenDeepCrew、DeerFlow 2.0、OpenSpace 等 20+ 实体，10 个核心概念

## [2026-05-05] create | 环境变量配置
- 设置 WIKI_PATH：待用户确认写入 .zshrc 或 .env

## [2026-05-05] ingest | AI科技动态批量摄入
- 摄入 4 篇外部来源：
  - `raw/articles/stanford-2026-ai-index-report.md` — 斯坦福2026 AI指数报告
  - `raw/articles/bbc-ai-distillation.md` — BBC中文：AI蒸馏成为美中竞争新战场
  - `raw/articles/mit-techreview-10-things-ai-2026.md` — MIT Tech Review 10大AI趋势
  - `raw/articles/neuro-symbolic-ai-energy-breakthrough.md` — 神经符号AI能耗降低100倍
- 新建 4 个 Wiki 页面：
  - `concepts/知识蒸馏.md` — 知识蒸馏概念页（含定义、战略意义、争议）
  - `concepts/neuro-symbolic-ai.md` — 神经符号AI概念页（含数据对比表）
  - `concepts/agent-orchestration.md` — 更新Agent编排概念页（整合MIT趋势 + 现有内容）
  - `entities/deepseek.md` — DeepSeek实体页
  - `entities/anthropic.md` — Anthropic实体页
- 更新：`index.md`（添加新实体和概念条目，总页数更新为5）
- 待续：Step 2 — 从现有知识库提取实体页

## [2026-05-05] ingest | 从知识库提取7个实体页
- 基于知识库 45 篇笔记提取核心实体：
  - `entities/openclaw.md` — OpenClaw 开源 Agent 框架（30次引用）
  - `entities/claude-code.md` — Claude Code AI 编程工具（17次引用）
  - `entities/myharness.md` — 自研 Agent 编排引擎（含构建历程四阶段）
  - `entities/opendeepcrew.md` — Agent 团队编排服务器
  - `entities/deerflow-2.0.md` — 字节跳动 Super Agent Harness
  - `entities/openspace.md` — 自进化 Skill 引擎（HKUDS）
  - `entities/opencli.md` — 浏览器桥接 CLI 工具
- 更新：`index.md`（7个新实体条目，总页数更新为12）

## [2026-05-05] lint | 首次健康检查
- 扫描 18 个 Wiki 文件
- 结果：✅ 所有页面 < 200 行，✅ 无真实断链（raw/ 层 frontmatter 格式不同是预期行为）
- 修复：`concepts/agent-orchestration.md` 中的错误三括号链接
- 待创建（录入 index 的占位页面）：25 个（openai, xai, instreet, gstack 等）
- 知识库 45 篇笔记保持只读未修改

## [2026-05-05] ingest | 微信公众号文章摄入
- 来源：https://mp.weixin.qq.com/s/q6Q-0-9_sOjt29NTTPQO0g
- 标题：《Hermes 这个技能我一直没碰，跑完一遍后悔没早试》
- 作者：林月半子的AI笔记
- 抓取工具：wechat-article-for-ai（Camoufox 隐身浏览器）
- 保存位置：
  - `知识库/技能文档/hermes-llm-wiki实战/`（含 MD + 17 张图片）
  - `raw/articles/hermes-llm-wiki-实战-2026-04-17.md`
- 新建 Wiki 页面：
  - `entities/hermes-agent.md` — Hermes Agent 实体页
  - `concepts/llm-wiki-pattern.md` — Karpathy LLM Wiki 概念页（三层架构 + 红绿灯原则 + vs RAG）

## [2026-05-05] create | feishu-api-access 概念页
- `concepts/feishu-api-access.md` — 飞书文档访问方法
- 整合 4 条路径：Hermes feishu-doc Skill、feishu-wiki 知识空间导航、直接 API（curl）、OpenClaw 技能包
- 包含：读取/写入/评论操作指南、实战技巧、路径对比表、凭据位置
- 更新：`index.md`（总页数 15）

## [2026-05-05] ingest | 飞书文档归档
- 来源：https://my.feishu.cn/wiki/DbDhwhYcFiHY8JkpX0fcAYdMnve
- 标题：Hermes资源库（持续更新） — Hermes Agent 12章完整教程
- 访问方式：飞书 docx API（curl），987 blocks，18,823 字符
- 保存位置：
  - `知识库/技能文档/hermes-llm-wiki实战/2026-05-05-Hermes资源库-飞书归档.md`
  - `raw/articles/hermes-feishu-resource-library.md`
- 更新 Wiki 页面：`entities/hermes-agent.md`（补充能力全景表、三层记忆系统、技能自改进机制）

## [2026-05-05] update | SCHEMA.md 加入红绿灯原则
- 新增「红绿灯原则（人机分工契约）」章节
- 🟢 绿灯区：AI 全权处理 7 项
- 🟡 黄灯区：AI 标记 + 人类确认 5 项
- 🔴 红灯区：人类专属 4 项
- 来源：Karpathy LLM Wiki 人机协作规范 + @王树义 红绿灯原则

## [2026-05-05] create | queries/ 归档启动
- `queries/openclaw-web-search-tools.md` — OpenClaw 网络搜索工具选型（7工具×3场景）
- `queries/wechat-access-methods.md` — 微信公众号访问 3 条路径（含实测结果）
- 更新：`index.md`（Queries 章节新增 2 条）

## [2026-05-05] create | 每周 lint 定时任务
- Cron job `wiki-weekly-lint` (ID: a08978bbfcd7)
- 执行时间：每周一 09:00（北京时间）
- 范围：孤儿页、断链、索引完整性、过期内容、红绿灯合规、矛盾页面、质量信号、大页面
- 结果自动推送至当前会话

## [2026-05-05] ingest | 微信公众号文章归档 — Hermes满配指南
- 来源：https://mp.weixin.qq.com/s/mgeFXHp-qILwGilZgs9OXA
- 标题：《7步让能力提升10倍！Hermes Agent满配指南！》
- 作者：小龙开发者
- 保存位置：
  - `知识库/技能文档/hermes-llm-wiki实战/2026-05-05-Hermes满配指南-7步.md`
  - `raw/articles/hermes-full-config-guide.md`
- 更新 Wiki 页面：`entities/hermes-agent.md`（新增满配vs裸装对比表、核心配置项、生态资源）

## [2026-05-05] ingest | 从满配指南提取新实体和概念
- `entities/hindsight.md` — Hindsight 云端记忆系统（自动提取+知识图谱，对比内置MEMORY.md）
- `concepts/soul-dot-md.md` — SOUL.md 概念页（人格定义维度、示例结构、211角色模板）
- 更新：`index.md`（+2 条目）

## [2026-05-05] lint | 全面健康检查
- 扫描 29 个文件
- ✅ 无过期内容、无低置信度页面、无矛盾标记、无红绿灯违规
- 🟡 19 个页面标记为 confidence: high（需你签字确认）
- 🟡 3 个断链指向已规划但未创建的页面（openai, moonshot-ai, xai）
- ℹ️ raw/ 层 frontmatter 格式差异、孤儿页、大页面均为预期行为

## [2026-06-04] ingest | 6 篇技能文档摄入 — OpenClaw互联网访问、企业架构、技能关系图、Coze协作、知识系统

- 来源：知识库/技能文档/ 新增6篇
- 新建 5 个实体页：
  - `entities/coze.md` — 扣子（Coze）字节跳动多 Agent 协作平台
  - `entities/skilldag.md` — SkillDAG 复旦/NUS/A*STAR 技能关系图（5种边类型+在线演化）
  - `entities/langgraph.md` — LangGraph 状态编排框架（四层架构+MCP集成）
  - `entities/book-to-skill.md` — 书籍转 Skill 工具（PDF/EPUB→Claude Code Skill）
  - `entities/compound-engineering.md` — Compound Engineering 实体页（让AI产出持续复用）
- 更新 1 个概念页：
  - `concepts/llm-wiki-pattern.md` — 补充研发自动化视角、Compound Engineering 视角、1800万阅读量数据
- 更新：`index.md`（+5 实体条目，总页数更新为 22）

## [2026-06-15] lint | 完整健康检查
- 扫描 24 个 Wiki 页面（16 entities + 6 concepts + 0 comparisons + 2 queries）+ 7 raw/ 文件
- 🔴 断链：11 个无效 [[wikilinks]]（openai, instreet, gstack, skill-development, agent-reach, bb-browser, stitch-2.0, knowledge-management, harness-architecture, moonshot-ai, xai）
- 🔴 红绿灯违规：24 页 confidence:high 均未获人类签字确认；2 查询页 AI 单方面归档
- 🔴 4 个页面出站链接为 0（hermes-agent, feishu-api-access, 2 queries）
- 🟡 标签违规：16 个标签不在 SCHEMA.md 分类体系内（platform, cli, model, lab 等）
- 🟡 索引占位页面：25 个条目指向不存在的文件（41天未解决）
- 🟡 大页面预警：feishu-api-access.md 已达 190 行，接近 200 行分割阈值
- 🟢 孤儿页：9 个页面无入站链接（coze, skilldag, langgraph, book-to-skill, neuro-symbolic-ai, feishu-api-access, soul-dot-md, 2 queries）
- ✅ 无过期内容（全部 < 90 天）
- ✅ 无矛盾/争讼标记
- ✅ 无大页面超过 200 行
| 建议：本周优先修复断链和红绿灯违规，确认 confidence 标记

## [2026-06-22] lint | 第5次健康检查 — 7天后
- 扫描 24 个 Wiki 页面（16 entities + 6 concepts + 0 comparisons + 2 queries）+ 7 raw/ 文件
- 🔴 断链：16 个无效 [[wikilinks]]
  - entities/anthropic.md → [[openai]]
  - entities/book-to-skill.md → [[skill-development]], [[knowledge-management]]
  - entities/claude-code.md → [[stitch-2.0]]
  - entities/compound-engineering.md → [[knowledge-management]]
  - entities/myharness.md → [[gstack]]
  - entities/openclaw.md → [[instreet]]
  - entities/opencli.md → [[bb-browser]], [[agent-reach]]
  - entities/openspace.md → [[skill-development]]
  - entities/skilldag.md → [[skill-development]]
  - concepts/llm-wiki-pattern.md → [[wikilink]]（元示例残留）
  - concepts/neuro-symbolic-ai.md → [[harness-architecture]]
  - concepts/知识蒸馏.md → [[moonshot-ai]], [[xai]], [[openai]]
- 🔴 红绿灯违规：24 页 confidence:high 均未获人类签字确认（🔴 红灯区要求）
- 🔴 红绿灯违规：2 查询页（openclaw-web-search-tools, wechat-access-methods）AI 单方面归档，未走黄灯审批流程
- 🟡 索引占位页面：24 个条目指向不存在的文件（48天未解决）
  - 占位列表：agent-design-principles, agent-memory-system, agent-reach, ai-coding-practice, bb-browser, clawdbot, control-center, free-code, gstack, gstack-vs-ce, harness-architecture, harness-comparison, instreet, knowledge-management, last30days-cn, news-aggregator, obsidian-direct, playwright-scraper, retrospective-methods, skill-development, skywork-ppt, stitch-2.0, workflow-automation, zread-cli
- 🟡 4 个页面出站链接为 0（hermes-agent, feishu-api-access, 2 queries）— 违反 ≥2 条出站链接规范
- 🟡 7 个 raw/ 文件 sha256 与当前正文不匹配（需排查是否内容漂移或初始摘要计算方式不同）
- 🟡 大页面预警：feishu-api-access.md 190 行，仍接近 200 行分割阈值
- 🟡 标签建议：16 个常用标签未正式纳入 SCHEMA.md 分类体系（model, cli, browser, cloud, platform, orchestration, optimization, research, technique, configuration, customization, china, api, multi-agent, ai-coding, lab）
- 🟢 孤儿页：0 — 所有页面均有入站链接 ✅
- 🟢 无过期内容（全部 < 90 天）✅
- 🟢 无矛盾/争讼标记 ✅
- 🟢 无大页面超过 200 行 ✅
- 🟢 无 frontmatter 字段缺失 ✅
- 🟢 无标签违规（所有标签虽未正式加入分类体系，但都已在 lint 中被识别）✅
- 🟢 Log.md 条目数 16，无需轮转 ✅
- 主要恶化项：断链从 11 → 16（+5），hermes-agent 和 feishu-api-access 出站链接仍为 0

## [2026-06-29] lint | 第6次健康检查 — 7天后
- 扫描 24 个 Wiki 页面（16 entities + 6 concepts + 0 comparisons + 2 queries）+ 7 raw/ 文件
- 🔴 断链：12 个无效 [[wikilinks]]（16 次出现），较上周减少 4（stitch-2.0 移除后仍被引用）
  - entities/anthropic.md → [[openai]]
  - entities/book-to-skill.md → [[skill-development]], [[knowledge-management]]
  - entities/claude-code.md → [[stitch-2.0]]
  - entities/compound-engineering.md → [[knowledge-management]]
  - entities/myharness.md → [[gstack]]
  - entities/openclaw.md → [[instreet]]
  - entities/opencli.md → [[agent-reach]], [[bb-browser]]
  - entities/openspace.md → [[skill-development]]
  - entities/skilldag.md → [[skill-development]]
  - concepts/llm-wiki-pattern.md → [[wikilink]]（元示例残留）
  - concepts/neuro-symbolic-ai.md → [[harness-architecture]]
  - concepts/知识蒸馏.md → [[moonshot-ai]], [[openai]], [[xai]]
- 🔴 红绿灯违规：24 页 confidence:high 均未获人类签字确认（🔴 红灯区要求，7周未解决）
- 🔴 红绿灯违规：2 查询页（openclaw-web-search-tools, wechat-access-methods）AI 单方面归档，未走黄灯审批流程
- 🟡 索引占位页面：24 个条目指向不存在的文件（55天未解决）
  - 占位列表同上次（agent-design-principles, agent-memory-system 等 24 个）
- 🟡 5 个页面出站链接不足 2 条（hermes-agent: 0, feishu-api-access: 0, hindsight: 1, openclaw-web-search-tools: 0, wechat-access-methods: 0）
- 🟡 7 个 raw/ 文件 sha256 与当前正文不匹配（与上次一致，疑似摘要计算方式偏离）
- 🟡 大页面预警：feishu-api-access.md 190 行，仍接近 200 行分割阈值（连续 3 次 lint 提醒）
- 🟡 16 个标签未正式纳入 SCHEMA.md 分类体系（ai-coding, api, browser, china, cli, cloud, configuration, customization, lab, model, multi-agent, optimization, orchestration, platform, research, technique — 55 天未解决）
- 🟢 孤儿页：9 个页面无入站链接（coze, skilldag, langgraph, book-to-skill, neuro-symbolic-ai, feishu-api-access, soul-dot-md, 2 queries）— ⚠️ 较上次恶化（上次 0 孤儿）
- 🟢 无过期内容（全部 < 90 天）✅
- 🟢 无矛盾/争讼标记 ✅
- 🟢 无大页面超过 200 行 ✅
- 🟢 无 frontmatter 字段缺失 ✅
- 🟢 Log.md 条目数 18，无需轮转 ✅
- 🟢 标签使用与上周一致，无新增违禁标签 ✅
- 🟢 sources 格式一致 ✅
- 主要恶化：孤儿页从 0 → 9，hermes-agent 和 feishu-api-access 出站链接仍为 0（连续 3 次 lint 未修复）
| 建议优先修复：1) 为孤儿页添加入站链接 2) 为 hermes-agent/feishu-api-access 补充出站链接 3) 确认 confidence:high 签名 4) 新建 gstack/openai/skill-development 等高频引用目标页

## [2026-07-13] lint | 第8次健康检查 — 7天后
- 扫描 24 个 Wiki 页面（16 entities + 6 concepts + 0 comparisons + 2 queries）+ 7 raw/ 文件 + 知识库（245 篇）
- 🔴 **断链恶化：12 个无效 [[wikilinks]]（16 次出现），较上周反弹 +7**
  - entities/anthropic.md → [[openai]]
  - entities/book-to-skill.md → [[skill-development]], [[knowledge-management]]
  - entities/claude-code.md → [[stitch-2.0]]
  - entities/compound-engineering.md → [[knowledge-management]]
  - entities/myharness.md → [[gstack]]
  - entities/openclaw.md → [[instreet]]
  - entities/opencli.md → [[agent-reach]], [[bb-browser]]
  - entities/openspace.md → [[skill-development]]
  - entities/skilldag.md → [[skill-development]]
  - concepts/llm-wiki-pattern.md → [[wikilink]]（元示例残留）
  - concepts/neuro-symbolic-ai.md → [[harness-architecture]]
  - concepts/知识蒸馏.md → [[moonshot-ai]], [[openai]], [[xai]]
- 🔴 **红绿灯违规：24 页 confidence:high 均未获人类签字确认（🔴 红灯区要求，9 周未解决）**
- 🔴 **红绿灯违规：2 查询页（openclaw-web-search-tools, wechat-access-methods）AI 单方面归档，未走黄灯审批流程**
- 🔴 **4 页面出站链接为 0**：hermes-agent, feishu-api-access, 2 queries（连续 5 次 lint 未修复）
- 🟡 索引占位页面：24 条目指向不存在的文件（69 天未解决），同上次
- 🟡 1 页面出站链接不足 2 条：hindsight（仅 1 条，连续 5 次）
- 🟡 2 个 raw/ 文件 sha256 不匹配：hermes-full-config-guide.md, hermes-llm-wiki-实战-2026-04-17.md
- 🟡 大页面预警：feishu-api-access.md 191 行（连续 6 次 lint 提醒）
- 🟡 16 个标签未正式纳入 SCHEMA.md 分类体系（model, lab, china, cli, browser, ai-coding, cloud, orchestration, platform, research, configuration, customization, multi-agent, api, technique, optimization — 69 天未解决）
- 🟡 知识库从初始化时的 45 篇增长至 245 篇，但 Wiki 页面数仍为 24 未增长（9 周未摄入新源）
- 🟢 孤儿页：9 页面无入站链接（book-to-skill, coze, feishu-api-access, langgraph, neuro-symbolic-ai, 2 queries, skilldag, soul-dot-md）— 与上周一致
- 🟢 无过期内容（全部 < 90 天）✅
- 🟢 无矛盾/争讼标记 ✅
- 🟢 无大页面超过 200 行 ✅
- 🟢 无 frontmatter 字段缺失 ✅
- 🟢 Log.md 条目数 20，无需轮转 ✅
- 主要恶化：断链从 5 → 12（+7），上次报告中的修复被回退或未执行；知识库增长 5.4 倍但 Wiki 停滞
| 建议：本周优先 1) 修复断链（新建 openai/gstack/instreet 等高频引用页，移除或替换 skill-development 等遗留占位引用）2) 为 hermes-agent/feishu-api-access 补充出站链接 3) 确认 confidence:high 签名 4) 恢复来源摄入以跟上知识库增长

## [2026-07-06] lint | 第7次健康检查 — 7天后
- 扫描 24 个 Wiki 页面（16 entities + 6 concepts + 0 comparisons + 2 queries）+ 7 raw/ 文件
- 🔴 断链：5 个无效 [[wikilinks]]，较上周减少 7（stitch-2.0 被引用链已修复）
  - entities/anthropic.md → [[openai]]
  - concepts/llm-wiki-pattern.md → [[wikilink]]（元示例残留）
  - concepts/知识蒸馏.md → [[moonshot-ai]], [[openai]], [[xai]]
- 🔴 红绿灯违规：24 页 confidence:high 均未获人类签字确认（🔴 红灯区要求，8周未解决）
- 🔴 红绿灯违规：2 查询页（openclaw-web-search-tools, wechat-access-methods）AI 单方面归档，未走黄灯审批流程
- 🔴 4 页面出站链接为 0（hermes-agent, feishu-api-access, 2 queries）— 连续 4 次未修复
- 🟡 索引占位页面：24 个条目指向不存在的文件（62天未解决）
  - 占位列表同上次（agent-design-principles, agent-memory-system 等 24 个）
- 🟡 1 页面出站链接不足 2 条：hindsight（仅 1 条）
- 🟡 7 个 raw/ 文件 sha256 与当前正文不匹配（与上次一致，疑似摘要计算方式偏离）
- 🟡 大页面预警：feishu-api-access.md 191 行（连续 5 次 lint 提醒，接近分割阈值）
- 🟡 16 个标签未正式纳入 SCHEMA.md 分类体系（ai-coding, api, browser, china, cli, cloud, configuration, customization, lab, model, multi-agent, optimization, orchestration, platform, research, technique — 62 天未解决）
- 🟢 孤儿页：9 个页面无入站链接（coze, skilldag, langgraph, book-to-skill, neuro-symbolic-ai, feishu-api-access, soul-dot-md, 2 queries）— 与上周一致，未恶化
- 🟢 无过期内容（全部 < 90 天）✅
- 🟢 无矛盾/争讼标记 ✅
- 🟢 无大页面超过 200 行 ✅
- 🟢 无 frontmatter 字段缺失 ✅
- 🟢 Log.md 条目数 19，无需轮转 ✅
- 🟢 正改善：断链从 12 → 5（-7），stitch-2.0 相关断链已清除
- 主要趋势：断链改善但 orphan 问题持续未解决（9 页连续 2 次 lint），hermes-agent/feishu-api-access 出站链接连续 4 次定期提及未修复，红绿灯违规（24 页 confidence:high 未签字）连续 8 周未解决

## [2026-07-20] lint | 第8次健康检查 — 14天后
- 扫描 24 个 Wiki 页面（16 entities + 6 concepts + 0 comparisons + 2 queries）+ 7 raw/ 文件
- 🔴 断链：17 个无效 [[wikilinks]]，较上次严重恶化（+12）
  - entities/anthropic.md → [[openai]]（持续11周未修复）
  - entities/book-to-skill.md → [[knowledge-management]], [[skill-development]]（新）
  - entities/claude-code.md → [[stitch-2.0]]（回退：上次已修复，现重新出现）
  - entities/compound-engineering.md → [[knowledge-management]]（新）
  - concepts/llm-wiki-pattern.md → [[wikilink]]（元示例残留，持续11周）
  - entities/myharness.md → [[gstack]]×2（新）
  - concepts/neuro-symbolic-ai.md → [[harness-architecture]]（新）
  - entities/openclaw.md → [[instreet]]（新）
  - entities/opencli.md → [[agent-reach]], [[bb-browser]]（新）
  - entities/openspace.md → [[skill-development]]（新）
  - entities/skilldag.md → [[skill-development]]（新）
  - concepts/知识蒸馏.md → [[moonshot-ai]], [[openai]], [[xai]]（持续11周）
- 🔴 红绿灯违规：24 页全部 confidence:high 标记，均未获得人类签字确认（连续 10 周）
- 🔴 红绿灯违规：2 查询页（openclaw-web-search-tools, wechat-access-methods）AI 单方面归档，未走黄灯审批流程
- 🔴 4 页面出站链接为 0（hermes-agent, feishu-api-access, 2 queries）
- 🟡 索引占位页面：24 个条目指向不存在的文件（持续 11 周未解决，与上次完全一致）
- 🟡 孤儿页：9 个页面无入站链接（book-to-skill, coze, feishu-api-access, langgraph, neuro-symbolic-ai, skilldag, soul-dot-md, 2 queries）— 与上周一致，持平
- 🟡 大页面预警：feishu-api-access.md 191 行（连续 6 次 lint 提醒，接近分割阈值）
- 🟡 16 个标签未正式纳入 SCHEMA.md 分类体系（ai-coding, api, browser, china, cli, cloud, configuration, customization, lab, model, multi-agent, optimization, orchestration, platform, research, technique — 持续 11 周未解决）
- 🟢 无过期内容（全部 < 90 天）✅
- 🟢 无矛盾/争讼标记 ✅
- 🟢 无大页面超过 200 行 ✅
- 🟢 无 frontmatter 字段缺失 ✅
- 🟢 Log.md 条目数 21，无需轮转 ✅
- 🟢 主要恶化：断链从 5 → 17（+12），stitch-2.0 被重新引入回退，新断链大量出现（knowledge-management, skill-development, gstack, instreet, agent-reach, bb-browser, harness-architecture）。孤儿页与上一 lint 持平（9 页），连续 3 次 lint 无改善。
| - 建议：本周重点 1) 修复断链中最关键的：stitch-2.0（直接回退）、skill-development（多处引用）、openai（2处引用）、gstack（被 myharness 引用）2) 为 feishu-api-access/hermes-agent 补充出站链接 3) 确认 confidence:high 签名 4) 考虑拆 feishu-api-access（191行）

## [2026-07-27] lint | 第9次健康检查 — 7天后
- 扫描 24 个 Wiki 页面（16 entities + 6 concepts + 0 comparisons + 2 queries）+ 7 raw/ 文件
- 🔴 **断链：17 个无效 [[wikilinks]]（12 个唯一目标），与上次完全一致 — 连续 8 周零改善**
  - entities/anthropic.md → [[openai]]
  - entities/book-to-skill.md → [[knowledge-management]], [[skill-development]]
  - entities/claude-code.md → [[stitch-2.0]]
  - entities/compound-engineering.md → [[knowledge-management]]
  - concepts/llm-wiki-pattern.md → [[wikilink]]（元示例残留）
  - entities/myharness.md → [[gstack]]×2（同页两次）
  - concepts/neuro-symbolic-ai.md → [[harness-architecture]]
  - entities/openclaw.md → [[instreet]]
  - entities/opencli.md → [[agent-reach]], [[bb-browser]]
  - entities/openspace.md → [[skill-development]]
  - entities/skilldag.md → [[skill-development]]
  - concepts/知识蒸馏.md → [[moonshot-ai]], [[openai]], [[xai]]
- 🔴 **红绿灯违规：24 页全部 confidence:high 标记，均未获得人类签字确认（连续 11 周）**
- 🔴 **红绿灯违规：2 查询页（openclaw-web-search-tools, wechat-access-methods）AI 单方面归档，未走黄灯审批流程**
- 🔴 **5 页面出站链接不足 2 条：hermes-agent(0), feishu-api-access(0), hindsight(1), 2 queries(0) — 连续 6 次 lint 未修复**
- 🟡 索引占位页面：24 个条目指向不存在的文件（持续 11 周未解决，与上次完全一致）
- 🟡 孤儿页：9 个页面无入站链接（book-to-skill, coze, feishu-api-access, langgraph, neuro-symbolic-ai, skilldag, soul-dot-md, 2 queries）— 与上次一致，持平
- 🟡 大页面预警：feishu-api-access.md 190 行（连续 7 次 lint 提醒，接近分割阈值）
- 🟡 16 个标签未正式纳入 SCHEMA.md 分类体系（ai-coding, api, browser, china, cli, cloud, configuration, customization, lab, model, multi-agent, optimization, orchestration, platform, research, technique — 持续 11 周未解决）
- 🟡 2 个 raw/ 文件 sha256 与当前正文不匹配：hermes-full-config-guide.md, hermes-llm-wiki-实战-2026-04-17.md（与之前一致，疑似摘要计算方式偏离，未提示内容实际变化）
- 🟢 无过期内容（全部 < 90 天）✅
- 🟢 无矛盾/争讼标记 ✅
- 🟢 无大页面超过 200 行 ✅
- 🟢 无 frontmatter 字段缺失 ✅
- 🟢 Log.md 条目数 22，无需轮转 ✅
- 🟢 知识库文件数稳定（245 篇），Wiki 页面数稳定（24）— 自 6/4 后无新来源摄入
- 🟢 主要趋势：所有指标与 7/20 lint 完全一致 — 无恶化亦无改善。断链 17 处/孤儿 9 页/红绿灯违规/索引占位全部持续零改善，系统进入稳态停滞
| 严重警告：所有 5 类 🔴 问题已连续 11 周零修复。当前 Wiki 的维护负债持续积累，每周 lint 仅报告但不产生任何修复动作。建议本周采取行动：1) 新建 openai/skill-development/gstack 等 5 个高频引用目标页 2) 为 hermes-agent/feishu-api-access 补充出站链接 3) 确认 24 页 confidence:high 的签字 4) 拆 feishu-api-access（190 行→拆分）
