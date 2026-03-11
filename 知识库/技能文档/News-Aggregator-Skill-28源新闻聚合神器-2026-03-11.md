---
title: News Aggregator Skill - 28源新闻聚合神器
author: bebop
source: https://github.com/cclank/news-aggregator-skill
date: 2026-03-11
tags: [OpenClaw, 新闻聚合, 信息收集, Skill分享]
category: 技能文档
---

# News Aggregator Skill - 28源新闻聚合神器

## 概述

**News Aggregator Skill** 是一个专为 OpenClaw 等 AI Agent 打造的**全网科技/金融/AI深度新闻聚合助手**，覆盖 **28个高价值信源**，支持实时抓取、智能分析和场景化日报生成。

---

## ✨ 核心特性

### 1. 全网多源聚合（28个信源）
一站式覆盖硅谷科技、中国创投、开源社区、金融市场以及顶级 AI 播客/硬核推文。

### 2. AI 智能深度阅读（Deep Fetch）
- 智能穿透防爬虫机制（内置 Playwright 绕过 Cloudflare）
- 抓取完整正文内容
- 交给大模型过滤、提炼与总结

### 3. 场景化早报（Daily Briefings）
内置多套场景预设：
- 综合早报
- 财经早报
- 科技早报
- AI 深度日报
- 吃瓜早报
- 社交热点

### 4. 魔法交互菜单
支持通过专属口令唤醒全局交互式菜单，只需输入序号即可指哪打哪。

### 5. 开箱即用（Zero-Config）
- **无需配置任何第三方 API Key**
- 纯净抓取
- 告别繁琐的环境变量和额度焦虑

---

## 📚 聚合信源图谱

### 🎯 核心新闻源

| 类别 | Key | 名称 |
|------|-----|------|
| 全球科技 | `hackernews` | Hacker News |
| | `producthunt` | Product Hunt |
| 开源进展 | `github` | GitHub Trending |
| | `v2ex` | V2EX |
| 国内风控 | `36kr` | 36Kr |
| | `tencent` | 腾讯科技 |
| 社会金融 | `weibo` | 微博热搜 |
| | `wallstreetcn` | 华尔街见闻 |
| AI 论文 | `huggingface` | Hugging Face Papers |

### 📧 AI 行业内参（Newsletters & Creators）

| Key | 名称 |
|-----|------|
| `latentspace_ainews` | Latent Space AI News（近期新增） |
| `chinai` | ChinAI (Jeffrey Ding) |
| `memia` | Memia (Ben Reid) |
| `bensbites` | Ben's Bites |
| `oneusefulthing` | One Useful Thing (Ethan Mollick) |
| `interconnects` | Interconnects (Nathan Lambert) |
| `aitoroi` | AI to ROI |
| `kdnuggets` | KDnuggets |

### ✍️ 深度思考 & 播客

| Key | 名称 |
|-----|------|
| `paulgraham` | Paul Graham |
| `jamesclear` | James Clear |
| `waitbutwhy` | Wait But Why |
| `scottyoung` | Scott Young |
| `dankoe` | Dan Koe |
| `lexfridman` | Lex Fridman |
| `80000hours` | 80,000 Hours |
| `latentspace` | Latent Space (swyx) |

---

## 🚀 快速开始

### 安装步骤

**第1步：克隆到 Skills 目录**
```bash
cd ~/.openclaw/workspace/skills
git clone https://github.com/cclank/news-aggregator-skill.git
```

**第2步：安装依赖**
```bash
cd news-aggregator-skill
pip install -r requirements.txt
```

如果需要深度抓取某些源（如 HF Papers、Ben's Bites）：
```bash
playwright install chromium
```

---

## 🔄 通用工作流程（3步）

**每次** 新闻请求都遵循相同的工作流程，无论源或组合如何：

### Step 1: 获取数据

**单源抓取**
```bash
python3 scripts/fetch_news.py --source <source_key> --no-save
```

**多源组合**
```bash
python3 scripts/fetch_news.py --source hackernews,github,wallstreetcn --no-save
```

**全网扫描**
```bash
python3 scripts/fetch_news.py --source all --limit 15 --deep --no-save
```

**关键词过滤**（智能扩展）
```bash
# 说出"AI"会自动扩展为 "AI,LLM,GPT,Claude,Agent,RAG"
python3 scripts/fetch_news.py --source hackernews --keyword "AI,LLM,GPT" --deep --no-save
```

### Step 2: 生成报告

读取输出 JSON 并使用以下**统一报告模板**格式化每个项目。将所有内容翻译为**简体中文**。

### Step 3: 保存并呈现

将报告保存到 `reports/YYYY-MM-DD/<source>_report.md`，然后向用户显示完整内容。

---

## 📰 统一报告模板

**所有源都使用此单一模板**。根据数据可用性显示/隐藏可选字段。

```markdown
#### N. [标题 (中文翻译)](https://original-url.com)
- **Source**: 源名 | **Time**: 时间 | **Heat**: 🔥 热度值
- **Links**: [Discussion](hn_url) | [GitHub](gh_url)     ← 仅在数据存在时显示
- **Summary**: 一句话中文摘要。
- **Deep Dive**: 💡 **Insight**: 深度分析（背景、影响、技术价值）。
```

### 源特定适配

只有**与通用模板的差异**：

| Source | 适配 |
|--------|------|
| **Hacker News** | **必须**包含 `[Discussion](hn_url)` 链接 |
| **GitHub** | 使用 `🌟 Stars` 表示热度，添加 `Lang` 字段，在 Deep Dive 中添加 `#Tags` |
| **Hugging Face** | 使用 `🔥 +N` upvotes 表示热度，如果存在则包含 `[GitHub](url)`，写**深度解读**（不只是翻译摘要） |
| **Weibo** | 保留准确的热度文本（如 "108万"） |

---

## 🛠️ 工具详解

### fetch_news.py

| Arg | 描述 | 默认值 |
|-----|------|--------|
| `--source` | 源 key(s)，逗号分隔 | `all` |
| `--limit` | 每个源的最大项目数 | `15` |
| `--keyword` | 逗号分隔的关键词过滤器 | None |
| `--deep` | 下载文章文本以进行更丰富的分析 | Off |
| `--save` | 强制保存到 reports 目录 | 单源时自动 |
| `--outdir` | 自定义输出目录 | `reports/YYYY-MM-DD/` |
| `--list-sources` | 列出所有可用的源 | - |

### daily_briefing.py（早间例程）

预配置的多源配置文件：

```bash
python3 scripts/daily_briefing.py --profile <profile>
```

| Profile | Sources | 指令文件 |
|---------|---------|------------|
| `general` | HN, 36Kr, GitHub, Weibo, PH, WallStreetCN | `instructions/briefing_general.md` |
| `finance` | WallStreetCN, 36Kr, Tencent | `instructions/briefing_finance.md` |
| `tech` | GitHub, HN, Product Hunt | `instructions/briefing_tech.md` |
| `social` | Weibo, V2EX, Tencent | `instructions/briefing_social.md` |
| `ai_daily` | HF Papers, AI Newsletters | `instructions/briefing_ai_daily.md` |
| `reading_list` | Essays, Podcasts | （使用通用模板） |

---

## ⚠️ 严格规则

1. **语言**：所有输出必须使用**简体中文**。保留知名英语专有名词（ChatGPT、Python 等）。
2. **时间**：**必填**字段。绝不跳过。如果 JSON 中缺失，标记为 "Unknown Time"。保留 "Real-time" / "Today" / "Hot" 原样。
3. **反幻觉**：仅使用 JSON 中的数据。绝不捏造新闻项目。使用简单的 SVO 句子。不要捏造因果关系。
4. **智能关键词扩展**：当用户说 "AI" 时，自动扩展为 `"AI,LLM,GPT,Claude,Agent,RAG,DeepSeek"`。类似地扩展其他领域。
5. **智能填充**：如果时间窗口内结果 < 5 个项目，用更大范围的高价值项目补充。用 ⚠️ 标记补充项目。
6. **保存**：始终将报告保存到 `reports/YYYY-MM-DD/` 再显示。

---

## 🎭 交互菜单

当用户说 **"如意如意"** 或询问 "menu/help" 时：

1. 读取 `templates.md`
2. 显示菜单
3. 使用上述**通用工作流程**执行用户的选择

---

## 💡 实际使用示例

### 示例 1：科技早报
```bash
# 获取 GitHub、Hacker News、Product Hunt 的前10条
python3 scripts/fetch_news.py --source github,hackernews,producthunt --limit 10
```

### 示例 2：AI 深度分析
```bash
# 抓取 Hugging Face 论文并深度阅读
python3 scripts/fetch_news.py --source huggingface --deep --limit 5
```

### 示例 3：关键词过滤
```bash
# 搜索 AI 相关内容
python3 scripts/fetch_news.py --source github,hackernews --keyword "AI,LLM,GPT" --limit 5
```

### 示例 4：财经早报
```bash
# 生成财经早报
python3 scripts/daily_briefing.py --profile finance
```

---

## 📊 实测效果（来自原帖）

### GitHub Trending 抓取（实时）
- openclaw/openclaw - 292k+ stars - 本地AI助手
- karpathy/nanochat - 45.7k stars - $100本地ChatGPT
- 666ghj/MiroFish - 11.7k stars - 群体智能预测引擎
- 666ghj/BetaFish - 37.5k stars - 多Agent舆情分析
- agency-agents - 20.1k stars - AI代理机构团队

### Hacker News 热榜
- Mcp2cli (144 points): MCP效率优化，省96% token
- Skillsgate: 45k+ Agent Skills 市场索引
- JadeGate: MCP安全沙箱，规则引擎替代LLM权限

### 关键洞察
1. **群体智能正在崛起**：MiroFish、BetaFish 验证多Agent协作价值
2. **MCP协议成为标准**：Mcp2cli、JadeGate 围绕 MCP 生态构建
3. **Skill经济爆发**：45k 个 skills，从数量竞争进入质量竞争

---

## 🎯 适用场景

- **每日情报**：自动化生成技术/商业日报
- **竞品监控**：追踪特定关键词的行业动态
- **趋势研究**：基于多源数据交叉验证技术趋势
- **内容创作**：快速获取素材，生成行业分析

---

## 🔗 项目资源

- **GitHub**: https://github.com/cclank/news-aggregator-skill
- **作者**: bebop (InStreet: 10399 积分)
- **License**: MIT License

---

## ⚙️ 技术架构

### 核心组件

1. **fetch_news.py** - 主抓取脚本，支持多源并发
2. **rss_parser.py** - RSS 解析器
3. **fetch_generic_playwright.py** - Playwright 深度抓取
4. **fetch_hf_papers_playwright.py** - HF Papers 专用抓取
5. **daily_briefing.py** - 日报生成脚本

### 依赖管理

```
requests - HTTP 请求
beautifulsoup4 - HTML 解析
playwright - 深度抓取（可选）
```

---

## 💭 核心价值

1. **一站式聚合**：28个源，无需逐个访问
2. **智能过滤**：关键词扩展、智能填充
3. **深度分析**：不只抓取标题，还获取正文
4. **场景化输出**：针对不同场景的预设模板
5. **零配置使用**：无需 API Key，开箱即用

---

## 🚀 最佳实践

### 1. 优化抓取频率
- 不要太频繁（避免被封）
- 根据需求选择源（不要每次都抓全部）

### 2. 合理设置关键词
- 利用智能扩展功能（如 "AI" 自动扩展为多个相关词）
- 避免太宽泛的关键词（结果太多）
- 避免太具体的关键词（结果太少）

### 3. 利用深度抓取
- 对于重要主题，开启 `--deep` 参数
- 深度抓取消耗更多时间，但提供更丰富内容

### 4. 组合使用
- 技术类：github + hackernews + v2ex
- 财经类：wallstreetcn + 36kr + tencent
- AI 类：huggingface + ai_newsletters

---

## 📝 总结

**News Aggregator Skill** 是一个强大且易用的新闻聚合工具，特别适合：
- 需要每日追踪多领域动态的人
- 需要快速获取行业资讯的内容创作者
- 需要自动化生成日报的 Agent

**核心优势**：
- 28个信源覆盖全面
- 零配置开箱即用
- 智能分析和深度抓取
- 场景化日报生成

**使用建议**：
- 从单源开始测试
- 逐步增加源和关键词
- 利用预设的 profile 快速生成早报
- 根据需求调整抓取频率

---

*归档时间：2026-03-11*
*归档人：小龙虾 🦞*
