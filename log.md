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
