---
title: OpenClaw 网络信息搜索工具选型
created: 2026-05-05
updated: 2026-05-05
type: query
tags: [tool, comparison, workflow, practice]
sources:
  - entities/opencli.md
  - entities/openclaw.md
  - 知识库/技能文档/2026-03-16-这两个开源利器，OpenCLI和BB-Browser，让你的OpenClaw瞬间拥有全互联网能力.md
  - 知识库/技能文档/News-Aggregator-Skill-28源新闻聚合神器-2026-03-11.md
  - 知识库/技能文档/Last30days-CN-中国平台30天深度研究技能-2026-04-01.md
  - 知识库/技能文档/Agent Reach - 微信公众号访问方法.md
  - 知识库/教程指南/Apify-Claude Code实时抓取全网数据.md
confidence: high
---

# OpenClaw 网络信息搜索工具选型

> 问题：OpenClaw 搜索网络信息可以使用什么工具？
> 归档：2026-05-05 | 基于 Wiki 知识库综合

## 一、浏览器桥接类（复用登录态，无需 API）

### OpenCLI — 44平台244命令
- `opencli zhihu search "keyword"` / `opencli bilibili hot -f json`
- 安装：`npm install -g @jackwener/opencli` + Chrome 扩展
- 优点：极简轻量，AI 加持下自动发现 API
- 缺点：依赖本地 Chrome，适配器可能因网站改版失效

### BB-Browser — 103命令×36平台
- `bb-browser site twitter/search "AI agent"` / `bb-browser site reddit/hot --openclaw`
- 安装：`npm install -g bb-browser`
- 支持 MCP 模式，可直接接入 Claude Code / Cursor
- 缺点：常驻 Chrome 吃资源，多代理并行易卡

## 二、专用搜索技能类（OpenClaw Skill）

### Last30days-CN — 中国8大平台深度研究
- 微博、小红书、B站、知乎、抖音、微信公众号、百度、今日头条
- 4 个源无需 API Key，中文 NLP 评分
- 安装：`github.com/ChiTing111/last30days-skill-cn`

### News Aggregator Skill — 28源新闻聚合
- 硅谷科技 + 中国创投 + 开源社区 + AI 播客
- 内置 Playwright 绕过 Cloudflare
- 支持场景化早报
- 安装：`github.com/cclank/news-aggregator-skill`

### Agent Reach — 13+平台（含微信公众号）
- `pip install https://github.com/Panniantong/agent-reach`
- 唯一专门支持微信公众号的路径

## 三、通用抓取引擎类

### Playwright Scraper — OpenClaw 原生 Skill
- 十大必装技能之一，处理动态页面

### Apify — 工业级确定性抓取
- "确定性抓取 + LLM 智能分析"分离架构
- 20 年积累的 Actor 生态

## 场景选型速查表

| 场景 | 推荐工具 |
|------|---------|
| 搜中文平台（小红书/知乎/B站） | Last30days-CN |
| 抓实时热榜/趋势 | OpenCLI |
| 微信公众号内容 | Agent Reach |
| 海外科技资讯监控 | News Aggregator |
| 深度抓取任意网站 | Playwright Scraper |
| 生产级批量抓取 | Apify |
| AI Agent 想直接操控浏览器 | BB-Browser + MCP |
