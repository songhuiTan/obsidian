# 小戴AI工作台（Personal AI Knowledge Workspace）— 本地优先、Agent 可调用的个人知识工作台

> 来源：GitHub 仓库调研 2026-08-07
> 仓库：[oyorf/person_dashboard](https://github.com/oyorf/person_dashboard)
> 配套 Skills：[oyorf/personal-workbench-skills](https://github.com/oyorf/personal-workbench-skills)
> 标签：AI, Agent, 个人知识库, Obsidian, 本地优先, 抖音数据, 社媒洞察, React, 隐私工程

---

## 项目定位

**一个本地优先、可被 AI Agent 调用并持续维护的个人知识库，以及它的可视化 Workbench。**

作者 oyorf（小戴）把 Raw 素材、Wiki 知识、阅读、灵感、内容生产、社媒研究和账号数据放进一套**可读的 Markdown 目录**（可直接用 Obsidian 打开）；Workbench 负责索引、搜索、阅读和图表展示。公开仓库只包含 synthetic demo，不包含真实知识库或账号数据。

| 维度 | 内容 |
|------|------|
| 语言 | JavaScript（React 19 + Vite 6） |
| Star/Fork | 12⭐ / 5 fork（2026-08-07 更新，活跃） |
| 许可证 | MIT |
| 运行要求 | Node.js 20+ |
| 核心哲学 | **不绑定专有数据库**，纯 Markdown；**真实数据和登录态留在本机** |

## 目录结构：两层独立理解

```text
person_dashboard/
├── 个人知识库/   # 独立的 Markdown Vault 与 synthetic demo 数据
└── Workbench/    # 前端、服务端、测试、模板和开发工具
```

- `个人知识库/`：Raw、Wiki、灵感、内容成果、抖音数据契约——用 Obsidian 或其他 Markdown 工具直接打开
- `Workbench/`：不内置个人内容；默认读取旁边的 `个人知识库/`，可通过环境变量 `PERSONAL_DASHBOARD_VAULT_ROOT` 连接其他 Vault

## 能做什么

1. 用 Markdown 和 Obsidian 管理个人知识库，**不绑定专有数据库**
2. Workbench 中查看 Raw、Wiki、知识关系（星图）、书架、灵感和内容状态
3. 展示脱敏的近期社媒风向与主题研究报告
4. 展示抖音创作者中心的作品、账号趋势、合集、留存和观众分析
5. **将需要推理或登录态的任务交给用户自己的 Codex、Claude Code 或其他 Agent**
6. 所有真实内容和登录态留在本机

## 技术栈

| 类别 | 选型 |
|------|------|
| 前端框架 | React 19.2 + Vite 6.4 + react-router-dom 7 |
| 知识图谱 | d3-force（星图）+ gsap + motion 动画 |
| 搜索 | minisearch（本地全文搜索） |
| 图表 | recharts |
| Markdown 处理 | unified + remark-parse + react-markdown + rehype-sanitize |
| 元数据 | gray-matter（frontmatter 解析） |
| 数据契约 | zod（schema 校验） |
| Excel 解析 | xlsx（抖音官方导出） |
| 文件扫描 | fast-glob |
| 服务端 | Node 原生（books/materials/vault-index/vault-sync/social-insights 等 .mjs 模块） |

## 架构拆解

### Workbench 服务端模块（server/）

- `vault-index.mjs` / `vault-sync.mjs` — Vault 索引与同步
- `materials.mjs` / `material-reading-state.mjs` — Raw 素材与阅读状态
- `books.mjs` — 书架与阅读进度
- `social-insights.mjs` — 社媒洞察
- `wiki-ingest-runner.mjs` — Wiki 摄入
- `codex-runner.mjs` — Codex 任务执行（Agent 集成点）
- `security.mjs` — 安全边界
- `public-config.mjs` — 公开配置

### 配套 Agent Skills（个人工作台 Skills 仓库）

1. **`research-social-insights`**：先用官方/一手来源确认事实，再从科技媒体、中文大众社媒、X 和海外社区观察讨论；区分事实/用户观点/反方声音/Agent 综合判断；排除昵称、头像、账号 ID 等个人信息。公开网页用 Agent 自带网络能力；需登录的来源在用户已授权的浏览器会话中真实导航，**不调用平台 API、不逆向、不绕登录**
2. **`douyin-account-data`**：只处理用户本人有权访问的抖音创作者中心；授权页面模拟正常切换与点击，**下载平台官方 Excel**，页面可见信息补充字段；解析后经一致性检查和质量门禁生成 `current.json`。**不调用抖音开放 API、不抓取他人账号、不读私信**

## Agent-first 设计（核心亮点）

这个仓库的设计是**为 Agent 而生**：

- README 明确指导："推荐把这个仓库直接交给支持代码和终端操作的 AI Agent（Codex、Claude Code、WorkBuddy）。让 Agent 负责克隆、安装依赖、启动项目和处理数据适配，你只需要把目标和数据需求讲清楚。"
- 提供完整的 Agent 指令模板（运行项目 / 接入自己的知识库）
- **AGENTS.md 定义了三重边界**：
  - **Public data boundary**：绝不复制真实 Vault/账号导出/评论/消息/浏览历史/凭据/本地路径/截图进公开仓库；demo 必须 synthetic 且明确标注；`wiki/` demo 只限通用 concepts/frameworks 页面
  - **Public feature boundary**：公开导航不暴露 Brainstorm/运行存档/公众号模块；默认 loopback-only，不暴露本地变更 API 给 LAN
  - **Release gate**：`npm test` + `npm run build` + `npm run privacy:scan`——隐私扫描拒绝个人用户名、home 目录路径、私有 Vault 名、凭据赋值、已知私有产品标识符

## 关键设计原则

1. **缺数据保持缺失，不用 0 或虚构数据顶替**——字段或口径对不上时页面保持缺失
2. **适配不做手动编码**：告诉 Agent 需求 → Agent 完成接口梳理、字段映射、数据转换、配置和验证
3. **数据契约驱动**：每个页面期待特定目录/字段/数据含义，通过 zod schema 校验
4. **隐私扫描自动化**：privacy:scan 覆盖整个仓库（含 Vault），发布门禁的一部分

## 开发验证

```bash
cd Workbench
npm install
npm run dev          # 运行
npm test             # 构建 + node --test 全量测试（18+ 测试文件）
npm run build
npm run privacy:scan # 隐私扫描
```

---

## 战略分析

**这个项目与用户现有栈（Hermes + Obsidian 知识库 + 自媒体运营）高度同构，是"知识工作台 + Agent"形态的极佳参考。**

| 维度 | person_dashboard | 用户当前体系 |
|------|-----------------|-------------|
| 知识存储 | Markdown Vault（Obsidian 可开） | Obsidian 知识库（README+快速导航+技能文档） |
| Agent 接入 | Codex/Claude Code 运行 + 配套 Skills | Hermes（gbrain 索引 + 53 Skills） |
| 社媒洞察 | research-social-insights Skill | InStreet 经验 / 公众号运营 |
| 账号数据 | douyin-account-data Skill（官方 Excel） | 暂无对应 |
| 隐私保护 | privacy:scan 发布门禁 | 未系统化 |
| 可视化 | React Workbench 星图/图表 | 无（纯 Markdown 导航） |

**核心启发（按价值排序）：**

1. **Agent-first 的仓库设计值得照搬**——README 直接给 Agent 指令模板、AGENTS.md 定义数据/功能/发布三重边界。用户运营的 Obsidian 知识库 + Hermes 生态可以借鉴这套"把仓库本身设计成 Agent 可操作单元"的思路（AGENTS.md 边界 + 可执行的验证命令）。

2. **privacy:scan 发布门禁是稀缺实践**——用户如果要公开分享知识库或技能，自动扫描个人用户名/路径/凭据的发布门禁比人工检查可靠得多。这是 Hermes 生态没有覆盖的能力。

3. **抖音数据分析的合规路径**——授权会话 + 官方 Excel 下载 + 质量门禁，不逆向不绕反爬，与技能库已归档的"抖音元数据提取 fallback"形成对比：合规优先的方案设计。用户做自媒体（InStreet 账号 xiaolongxia_1177）如果要做账号数据分析，这条路径可直接参考。

4. **"缺数据保持缺失"原则**与知识库归档流程的"不虚构"精神一致——归档时缺图片标"(图片缺失)"而不是编造，同源。

5. **两层分离架构**（内容 Vault 与工具 Workbench 独立）与 kb-builder 的双 Repo 分离、用户 Obsidian 的"内容/索引分离"同构——验证了该模式在个人知识管理中的普适性。

**局限**：12⭐ 小众项目；前端较重（React + d3 + gsap）；公开仓库只有 synthetic demo，真实能力依赖配套 Skills 仓库的成熟度。

## 相关资源

- 仓库：https://github.com/oyorf/person_dashboard
- 配套 Skills：https://github.com/oyorf/personal-workbench-skills
- 关联归档：kb-builder（Claude Code Skill + ChromaDB 知识库）、Trove AI（Obsidian 本地知识库）、Open-Notebook（NotebookLM 替代品）

## 归档日志

- 2026-08-07 归档（GitHub API + README/AGENTS.md 分析，clone 超时改用 API）
