---
title: Agent 访问微信公众号内容的3条路径
created: 2026-05-05
updated: 2026-05-05
type: query
tags: [tool, integration, practice, tutorial]
sources:
  - 知识库/技能文档/Agent Reach - 微信公众号访问方法.md
  - 知识库/技能文档/Last30days-CN-中国平台30天深度研究技能-2026-04-01.md
  - 知识库/技能文档/OpenClaw爆文分析与养虾实战-2026-03-11.md
confidence: high
---

# Agent 访问微信公众号内容的3条路径

> 问题：如何使得 agent 能够访问微信公众号的内容？
> 归档：2026-05-05 | 3条路径实测

## 路径一：Agent Reach + wechat-article-for-ai（推荐，抓取指定文章）

```bash
# 1. 安装
pip install https://github.com/Panniantong/agent-reach/archive/main.zip
pip install 'camoufox[geoip]' markdownify beautifulsoup4 httpx mcp miku_ai

# 2. 克隆工具
mkdir -p ~/.agent-reach/tools
cd ~/.agent-reach/tools
git clone https://github.com/bzd6661/wechat-article-for-ai.git

# 3. 抓取文章
cd ~/.agent-reach/tools/wechat-article-for-ai
python3 main.py "https://mp.weixin.qq.com/s/文章ID"
```

首次运行自动下载 Camoufox 隐身浏览器（298MB）。输出 Markdown + YAML frontmatter，图片自动下载到本地。

MCP 集成（可选）：
```bash
npm install -g mcporter
mcporter config add wechat --command 'python3' --args '["mcp_server.py"]' --cwd ~/.agent-reach/tools/wechat-article-for-ai
mcporter call 'wechat.convert_article(url: "https://mp.weixin.qq.com/s/文章ID")'
```

实测结果：2026-05-05 成功抓取《Hermes 这个技能我一直没碰，跑完一遍后悔没早试》，17张图片自动下载。

## 路径二：Last30days-CN（搜索微信公众号内容）

搜索近30天微信公众号上的相关内容，而非抓取单篇文章。
- 安装：`github.com/ChiTing111/last30days-skill-cn`
- 配置 WECHAT_API_KEY（可选）

## 路径三：OpenClaw Playwright + UA伪装（临时抓取）

OpenClaw 自主修改 UA 头伪装 Chrome，模拟真人访问路径，直接请求文章 URL。
- 优点：无需额外安装
- 缺点：反爬升级可能失效

## 路径对比

| 维度 | Agent Reach | Last30days-CN | Playwright |
|------|------------|---------------|------------|
| 适合场景 | 抓指定单篇 | 搜索微信公众号内容 | 临时抓取 |
| 反爬能力 | Camoufox隐身浏览器 | API驱动 | UA伪装 |
| 图片下载 | 自动 | 不处理 | 需手动 |
| 稳定性 | 高 | 中 | 中 |
| 配置复杂度 | 中 | 低 | 低 |
