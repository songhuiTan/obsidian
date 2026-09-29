---
type: concept
title: Kuhakucai 创作者工具三件套：douyin-mcp / Oula-Skills / Creator-Pro
date: '2026-09-25'
source: Kuhakucai（欧拉欧拉）· GitHub
source_url: 'https://github.com/Kuhakucai'
ingested_via: put_page
ingested_at: '2026-09-25T01:23:25.281Z'
source_kind: put_page
tags:
  - 创作者工具
  - 开源
  - 抖音
  - Agent
  - AI
  - MCP
  - Skill
---

# Kuhakucai 创作者工具三件套 — 从开源 MCP 到闭源桌面版的完整创作者数据栈

> 来源：Kuhakucai（GitHub 名，即"欧拉欧拉"，与 Puppetsho 为同一作者的提交身份）｜GitHub
> 仓库一：[Kuhakucai/douyin-mcp](https://github.com/Kuhakucai/douyin-mcp)（76⭐，AGPL-3.0）
> 仓库二：[Kuhakucai/Oula-Skills](https://github.com/Kuhakucai/Oula-Skills)（16⭐）
> 仓库三：[Kuhakucai/Creator-Pro](https://github.com/Kuhakucai/Creator-Pro)（1⭐，闭源分发）

![Creator Pro 主界面](../assets/2026-09-25-Kuhakucai创作者工具三件套/creator-pro-preview.png)

## 概述

同一位作者围绕"内容创作者 + AI Agent"这一个命题铺了三层产品：**douyin-mcp**（开源 MCP Server，把抖音创作者中心数据喂给任意 Agent）、**Oula-Skills**（通用 Agent Skill 收藏库，方法论沉淀）、**Creator-Pro**（Windows 闭源桌面版，douyin-mcp 的产品化封装 + 许可证收费）。三者合成一条完整路径：开源 MCP 引流 → Skill 库立人设 → 桌面版商业化。

三个仓库的定位一句话：
- **douyin-mcp**：本地运行的抖音创作者数据 MCP Server——让 AI 同时读懂"你的抖音数据"（播放/完播/互动等指标）和"视频内容"（音轨 ASR 文案），做有证据的复盘。
- **Oula-Skills**：作者自用验证过的 5 个 Agent Skill（拷问式追问、PRD 生成、竞品分析、四阶段开发流、Skill 工厂）。
- **Creator-Pro**：v0.2.0 Windows 桌面工具，同步自己抖音作品 + 导入外部参考视频，通过 Agent Bridge 以只读方式把资料库提供给 Agent。

## douyin-mcp：架构拆解（三件套的技术核心）

### 数据流

```text
抖音创作者中心（指标 + 公开视频音轨）→ Playwright 专用 Chrome → 本地 SQLite → FastMCP → AI Agent
```

三层分离：浏览器层（Playwright 只读页面真实可见内容，不能绕过登录/权限/风控）→ 数据层（规范化为作品、指标快照、同步任务、质量状态存 SQLite）→ MCP 层（FastMCP 暴露 22 个工具 = 13 个浏览器数据工具 + 9 个文案流水线工具，Agent 不碰 Cookie 不读 DOM）。

### 项目结构（src/douyin_creator_mcp/，57 文件）

| 模块 | 职责 |
|------|------|
| `browser/` | session（持久化 profile）/ executor（唯一同步所有者线程）/ extractors（列表/详情提取）/ profile_lock（跨进程锁）/ media_observer |
| `services/` | browser_service（同步查询对比复盘导出）/ transcript_coordinator（持久后台 job+lease+恢复）/ transcript_policy（预热+增量+按需补齐）/ metrics（派生指标与排序公式） |
| `storage/` | SQLite + 有序带校验和的不可变迁移（3 个 migration）+ run/job/asset/revision 事务仓储 |
| `content/` | media.py（受控下载+FFprobe）/ asr.py（FFmpeg + 本地 faster-whisper） |
| `tools/` | browser_tools（13 工具契约）+ transcript_tools（9 工具契约） |

### 技术栈

| 层 | 选型 |
|----|------|
| 语言 | Python 3.11+ |
| MCP 框架 | FastMCP >=3.4 |
| 浏览器自动化 | Playwright >=1.45（项目专用 Chrome profile，不用日常浏览器） |
| 存储 | 本地 SQLite（作品/指标快照/同步任务/文案 run-job-revision） |
| ASR（可选） | faster-whisper（本地模型，运行时不联网下载）+ FFmpeg/FFprobe |
| 平台 | macOS / Windows 10-11，本机运行、单用户、单抖音账号 |
| 许可证 | AGPL-3.0-only |

### 核心设计亮点（值得借鉴的部分）

1. **文案按需加载（lazy ingestion）**：首次同步只入库元数据 + 后台预热最近 5 条；分析时缺文案才自动建 run（返回 `run_id`，Agent 轮询到 `analysis_ready` / `no_speech` / `failed`）。避免首次启动就全量 ASR 拖死体验。
2. **证据链完备**：每个结论带采集时间、缓存新鲜度、字段覆盖率、缺失原因、质量警告；页面没显示的值存 `null` 不补零；派生比率只用同一快照的分子分母并记录公式版本。
3. **账号指纹绑定**：首次同步后基于作品标题+发布时间摘要建不可逆指纹，检测到误切账号返回 `account_mismatch` 拒写——不存昵称/原始标题/作品 ID。
4. **安全边界干净**：MCP 不返回 Cookie/localStorage/验证码/签名媒体 URL/绝对路径；profile 跨进程锁；`purge --yes` 一次性清除全部本地数据。
5. **合规姿态前置**：强制 `acknowledge-platform-risk --yes` 才能用；README 置顶非官方声明，逐条列出抖音用户协议 2.4/5.1/5.3/7.1 的风险条款；AGPL 声明不授予平台数据权。

### 典型用法

```text
检查我的抖音数据状态。只在缓存过期时更新；然后按最近 30 天比较完播率、
5 秒完播率和互动率，给出复盘结论。每条结论说明数据时间、覆盖率、缺失项和证据。
```

安装即"对 Agent 说一句话"：`帮我克隆并安装配置 https://github.com/Kuhakucai/douyin-mcp.git`，README 专门写了 Agent 安装协议（7 步，含风险确认门禁），还有 BEGINNER_AGENT_GUIDE.md 给零基础用户。

## Oula-Skills：5 个方法论 Skill

| Skill | 一句话 | 与 Hermes 现有栈对照 |
|-------|--------|---------------------|
| **grill-me** | 行动前一次只追问一个关键决策、带推荐答案，决策树式消除歧义 | 对标 clarify 工具/Hermes 澄清模式，但"一次一问+给推荐"的交互纪律更严格 |
| **prd-generator** | 问答式补齐产品信息→结构化 PRD（含异常场景/竞品/验收标准） | Hermes 无专职 PRD skill，可补位 |
| **competitive-analysis** | PEST/商业模式画布/KANO/SWOT 框架生成证据驱动竞品报告 | 与 last30days-cn/open-source-research 互补（它是分析框架，不是数据源） |
| **verified-dev-flow** | 计划→评审计划→实施→评审实施四阶段闭环，阶段间用文档传递，审查不过不强制放行 | 对标 subagent-driven-development + writing-plans + plan 的组合；它的"审查未通过不放行"门禁值得借鉴 |
| **skill-creator** | 标准 Skill 目录初始化+校验脚本 | Hermes/GBrain 已各有 skill-creator，功能重叠 |

收录原则写得很克制："我自己实际使用、任务边界清晰、过程可验证可追溯、不用漂亮话代替证据"——这是把 Skill 当个人经验资产管理，与知识库"归档即资产"的思路同源。

## Creator-Pro：产品化与商业模式

v0.2.0（2026-09-15 发布），Windows 10/11 x64 安装包，仓库只有 README + 一张截图——**代码闭源，走 GitHub Releases 分发**：

- 功能 = douyin-mcp 的超集收敛：同步自己作品数据 + 导入外部参考视频（需声明处理权依据）+ 本地资料库 + **Agent Bridge**（`CreatorPro.AgentBridge.exe`，桌面软件运行时 Agent 可连，只读，不碰数据库不自动触发同步/下载/删除）。
- 商业模式：免费额度 + 本地许可证授权（付费提升外部参考解析额度）。
- 隐私姿态延续 douyin-mcp：数据全本地、专用浏览器 profile、Agent 读取需单独确认、未采集字段不补零。
- 未签名安装包触发 SmartScreen 提示，README 教用户校验 SHA256。

作者路径清晰可见：douyin-mcp（AGPL 开源建信任）→ Creator-Pro（闭源收费做变现），同一套数据管道的两种包装。

## 关键洞察

- **"创作者数据 + Agent"是一条正在成型的赛道**：TokBrain（08-18 归档，多模态知识库）、小戴AI工作台（08-07，星图+社媒洞察）、MediaCrawler（采集层）到 douyin-mcp（复盘层），共同点是"本地优先 + 证据链 + 权限感知"。
- **douyin-mcp 最值钱的不是功能是数据纪律**：null 不补零、快照分源、比率同源、公式版本化、覆盖率透明——这些是"AI 结论可信"的地基，比堆功能难得多。
- **README 即 Agent 协议**：douyin-mcp 把"Agent 应该怎么装我"写成 7 步协议（含风险确认门禁、交接话术、3 个示例请求），这是 Agent 时代 README 写法的新范式——文档的第一读者已经是 Agent 而不是人。
- **开源→闭源的分层变现**：管道开源（AGPL 防白嫖云服务）+ 桌面产品闭源收费，个人开发者的现实路径。

## 战略分析

### 与现有工具链的对照

| 维度 | douyin-mcp | 现有栈（Hermes + 知识库） |
|------|-----------|--------------------------|
| 抖音数据获取 | 创作者中心指标（本人账号，Playwright） | z-video-downloader（下载）、kb-archive-article 的 douyin-tiktok-scraper（单视频解析）、MediaCrawler 归档（多平台爬取） |
| 抖音内容理解 | 音轨 ASR（faster-whisper 本地） | video 转录栈（whisper 同源） |
| 复盘能力 | 指标对比+证据链（22 个 MCP 工具） | 无——现有栈能"看内容"但不能"看指标" |
| 平台 | macOS ✅ / Windows ✅ | TokBrain 是 Windows-only，douyin-mcp 补上 macOS |

### 差距与借鉴点

- **现有栈缺"创作者本人指标"这一层**：目前所有抖音相关工具都是内容侧（下载/转录/归档），没有经营数据侧（完播率/互动率/涨粉）。若做自己的账号复盘，douyin-mcp 是现成缺口填补，且支持 macOS。
- **证据链设计可直接搬**：null 不补零、快照分源、派生指标记录公式版本——这套纪律适用于任何"AI 读数据下结论"的场景，包括 InStreet 发帖数据复盘。
- **风险要正视**：非官方工具，Playwright 自动化访问可能触发抖音风控（README 自己反复警告）；76⭐ 单作者项目，DOM 变更即失效的维护风险真实存在；AGPL 传染性对二次开发有约束。

### 整合可能性

- **低门構试用**：uv/venv 装 douyin-mcp，`doctor` 通过后扫码登录，让 Hermes 通过 native-mcp 接入，先跑"最近 30 天互动率 Top5 + 共同点"这一个查询验证价值。
- **Oula-Skills 按需摘取**：verified-dev-flow（四阶段门禁）和 grill-me（拷问式澄清）可以单独复制进 Hermes skills 目录试用，与现有 plan/clarify 流程互补而非替换。
- **Creator-Pro 暂不适用**：Windows-only + 闭源 + 1⭐，观察即可。

## 术语表

- **Agent Bridge**：Creator-Pro 的只读接口进程，桌面软件运行时 Agent 可查询资料库，不能触发写操作。
- **revision（文案）**：不可变的文案版本单元，分页游标固定到 revision，进程重启后仍有效。
- **预热（warmup）**：首次同步后后台只 ASR 最近 5 条缺文案视频的策略，避免全量回溯。
- **账号指纹**：基于作品标题+发布时间摘要的不可逆哈希，用于检测误切账号，不存个人信息。

## 相关文档

- [[技能文档/2026-08-18-TokBrain-抖音内容本地多模态知识库]] — 内容侧对照（Windows-only）
- [[技能文档/MediaCrawler-55K-Star多平台数据采集开源工具]] — 采集层对照
- [[技能文档/2026-08-07-小戴AI工作台-personal-ai-workbench-本地优先Agent知识工作台]] — 同为本地优先创作者工作台
- [[技能文档/2026-07-25-Matt-Pocock-loop-me-Skill-拷问27轮转自动化工作流]] — grill-me 类拷问式 Skill 对照

## 归档日志

- 2026-09-25 归档。用户微信分享三仓库链接并标注"研究归档"。GitHub API 核实三仓库（76⭐/16⭐/1⭐，AGPL-3.0，douyin-mcp 2026-07-07 创建、07-29 最后 push，Creator-Pro v0.2.0 release 2026-09-15）；本机 git clone 被网络拦截，全程走 GitHub REST API + raw.githubusercontent.com 读取（README×3 + pyproject.toml + 5 个 SKILL.md + 文件树），无本地克隆。图片 3 张自 raw 通道下载（analyze/update/creator-pro 截图）。
