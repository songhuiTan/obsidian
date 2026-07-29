---
source: 微信公众号
author: 拾码备忘录
account: 拾码备忘录
date: 2026-07-06
url: https://mp.weixin.qq.com/s/0nWthyouIE56WHwBFR284A
tags: MediaCrawler, 数据采集, 爬虫, 小红书, 抖音, B站, 微博, 开源工具, Playwright
---

# MediaCrawler：55K Star 多平台数据采集开源工具

> GitHub: https://github.com/NanmiCoder/MediaCrawler（55K Star）
> 一款基于 Playwright 的多平台数据采集工具，覆盖小红书、抖音、快手、B站、微博、贴吧、知乎 7 个平台。

## 它能干什么

扫码登录后，输入关键词或指定帖子ID，自动抓取内容并导出为 Excel。支持 WebUI，无需命令行。

**支持的平台与数据范围：**

| 平台 | 可抓取内容 |
|------|-----------|
| 小红书 | 笔记 + 评论，关键词搜索或指定帖子ID |
| 抖音 | 视频 + 评论，支持创作者主页 |
| B站 | 视频信息 + 弹幕 + 评论 |
| 快手 | 视频 + 评论 |
| 微博 | 帖子 + 评论 + 转发 |
| 贴吧 | 帖子 + 楼层回复 |
| 知乎 | 问答 + 文章 + 评论 |

## 55K Star 的关键原因

1. **免破解签名算法** — 使用 Playwright 模拟真浏览器，扫码登录后直接利用登录态请求数据，无需研究 JS 加密
2. **WebUI 操作** — 打开网页 → 选平台 → 扫码 → 输关键词 → 点开始，对普通人最友好
3. **直接导出 Excel** — 自动格式化、自动列宽，配合 Excel 透视表就能做分析

## 实用场景

- **内容备份** — 输入自己的 ID，一键导出所有发过的内容
- **竞品调研** — 抓取某产品的数百条笔记和评论，词云分析口碑
- **社媒研究** — 传播学/市场营销论文的数据收集
- **素材收集** — 攻略系列等批量内容一键入库

## 安装使用

官方推荐使用 `uv` 管理依赖（Python 包管理工具，比 pip 快一个量级）：

```bash
git clone https://github.com/NanmiCoder/MediaCrawler.git
cd MediaCrawler
uv sync
```

**运行前准备：**
1. 确保装了最新版 Chrome
2. 地址栏输入 `chrome://inspect#remote`，勾选「Allow remote debugging」
3. 看到页面显示 `Server running at: 127.0.0.1:9222`
4. 浏览器打开 `http://127.0.0.1:5173/` 即可看到 WebUI 界面

> 💡 可以直接把项目链接丢给 Codex / Claude Code 等 Agent 来自动配置部署。
