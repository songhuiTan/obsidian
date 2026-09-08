# z-video-downloader：把视频下载做成万能 Skill

> 来源：老章很忙｜Ai学习的老章 2026-06-23
> 原文：[我把视频下载做成了一个 Skill，很万能，不小心开源了](https://mp.weixin.qq.com/s/qFY0Ba8tZmpvsZrQO55A5A)
> GitHub：[tjxj/z-skills](https://github.com/tjxj/z-skills)（z-video-downloader 子目录）241⭐
> 标签：AI, Skill, 视频下载, yt-dlp, 开源, 工具

---

## 概述

老章将视频下载能力从 `z-web-pack Skills` 中独立提取并强化，开源为 `z-video-downloader` Skill。基于 yt-dlp 但解决了各种被拦截问题，一句话即可完成视频下载。

## 核心功能

### URL 自动识别

自动判断输入是直链、m3u8、YouTube、B 站还是 yt-dlp 支持的主流视频平台，无需手动指定来源。

### 下载策略

| 策略 | 说明 |
|------|------|
| 默认稳妥模式 | 单视频优先，避免播放列表填满硬盘 |
| 失败报告 | 失败时记录原因和兜底路径到报告，方便复盘 |
| 格式支持 | mp4 / webm / mov / mkv / m4v / flv / ogv |
| 流媒体支持 | m3u8 / mpd |

### 输出结构

```
Video/Downloads/YYYY-MM-DD-主题/
├── 下载的视频文件.mp4
├── 下载的视频文件.info.json
├── download-report.md
└── download-report.json
```

### 后续能力链

视频下载完成后，继续进入 AI 处理管线：

```
视频链接 → 本地 MP4 → transcript → 关键帧 → 多模态大模型分析视频 → 学习网页
```

视频总结 Skill 仍在调试中，与本文的下载 Skill 形成管线上下游。

## 战略分析

| 维度 | z-video-downloader | 当前 Hermes 视频管道 |
|------|-------------------|-------------------|
| 下载引擎 | yt-dlp（强化反拦截） | video_transcriber（yt-dlp + bilibili-api + douyin-scraper） |
| URL 识别 | 自动判断直链/m3u8/平台 | detect_video_platform() 三平台 |
| 下载控制 | 稳妥单视频优先，防满盘 | 无此策略 |
| 输出报告 | download-report.md + json | 无报告 |
| 下游管线 | → 转录 → 关键帧 → 多模态分析 → 学习网页 | → whisper ASR → 归档 |
| 配套 UI | 本地/服务器 UI 下载控制台 | 无 |
| 开源状态 | 已开源 | 已部署 |

z-video-downloader 的核心价值不在下载本身（yt-dlp 已经很强），而在 **对下载体验的产品化封装**：URL 自动分类、稳妥策略防翻车、失败报告可复盘、输出结构规范、以及"下载不是终点"的下游管线设计。这些产品化细节正是当前 Hermes 视频管道可以借鉴的。

## 归档日志

- 2026-07-30 归档
