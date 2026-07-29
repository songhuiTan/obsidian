---
title: "一个可以让 Codex 浏览视频的神级 Skill，我装完就没卸过"
source: "丛林｜极客之家"
source_url: "https://mp.weixin.qq.com/s/tLawqc_waEMhHEM2jtujqQ"
date: "2026-07-21"
tags: [AI, Skill, Claude Code, Codex, 视频分析, 开源, Agent]
---

# claude-video — 为 Agent 系统补上视频分析能力

## 概述

claude-video 是一个为 Claude Code / Codex 等 Agent 系统提供视频分析能力的 Skill，核心解决「Agent 能读网页、跑脚本、翻仓库，唯独看不了视频」的痛点。通过 yt-dlp + ffmpeg + Whisper 管线实现视频下载、抽帧、字幕转录和视觉理解，让 Agent "真看过"视频画面而不是只靠标题猜。GitHub 9300+ Star，纯标准库实现（7 个 Python 脚本 + 1 份 SKILL.md，无第三方依赖）。

开源地址：[bradautomates/claude-video](https://github.com/bradautomates/claude-video)

## 核心原理

管线分四步：

1. **yt-dlp 下载视频** — 支持 YouTube、TikTok、Loom、X、Instagram，本地 mp4/mov/mkv/webm 文件直接读
2. **ffmpeg 抽帧** — 按时长自动计算帧数，默认压成 512px 宽的 JPEG
3. **字幕优先，Whisper 兜底** — 有外挂/自动字幕的直接抓取，无字幕时转音频送 Whisper 转录
4. **帧 + 带时间戳的字幕交给 Claude** — 逐帧读完再回答

## 核心功能

### /watch 一条命令看视频

```
/watch https://youtu.be/xxx 第30秒发生了什么？
/watch ~/Movies/screen-recording.mp4 界面在哪一步崩的？
```

### 抽帧预算控制

| 视频时长 | 默认帧数 | 效果 |
|:--|:--|:--|
| 30秒内 | ~30帧 | 很密，关键时刻全覆盖 |
| 30秒到1分钟 | ~40帧 | 依然很密 |
| 1到3分钟 | ~60帧 | 够用 |
| 3到10分钟 | ~80帧 | 稀一些，能用 |
| 10分钟以上 | 100帧（上限） | 稀疏扫描，建议聚焦模式 |

两条硬上限：每秒最多 2 帧，总数最多 100 帧。帧默认 512px 宽 JPEG。

### 字幕优先，Whisper 兜底

- 视频自带字幕（人工/自动）→ 直接抓取，不花钱，不需配置 key
- 无字幕 → 音频转单声道 16kHz，送 Whisper
- 推荐 Groq 的 `whisper-large-v3`（便宜快），OpenAI `whisper-1` 当备选
- key 写在 `~/.config/watch/.env`，权限 0600
- `--no-whisper` 只看帧不听声音

### 聚焦模式（长视频）

```
/watch https://youtu.be/abc --start 2:15 --end 2:45
/watch video.mp4 --start 1:12:00
```

圈定时间段后帧密度更高，仍受每秒 2 帧上限约束。

### 分辨率调整

默认 512px 看画面够，读 PPT 文字/终端命令吃力时加 `--resolution 1024` 翻倍帧宽度。

其他常用参数：`--max-frames`（压帧数省 token）、`--fps`（手动指定帧率）、`--out-dir`（指定工作目录）。

## 安装方式

**Claude Code 用户：**
```
/plugin marketplace add bradautomates/claude-video
/plugin install watch@claude-video
```

**Codex 用户：**
```
git clone https://github.com/bradautomates/claude-video.git ~/.codex/skills/watch
```

**claude.ai 网页版：** 下载 release 里的 `watch.skill`，到 Settings → Capabilities → Skills 里添加，需先开启 Code execution。

首次 `/watch` 会自动检查依赖，macOS 上缺 ffmpeg/yt-dlp 自动 brew 装，Linux/Windows 打印安装命令。

## 适用场景

- **拆爆款视频**：问开头 3 秒画面、钩子设计，不用自己边看边记
- **看 bug 录屏**：定位到具体帧，描述画面状态，很多情况不用点开视频
- **给长视频脱水**：拉字幕出总结，重要地方再回原视频核对

## 关键洞察

作者认为这个项目最值钱的地方不在功能，在思路——Claude 缺视频输入，没等官方，拿 yt-dlp+ffmpeg 两个老工具拼了一条管线。一个人两个半月做到 9300+ Star。Skill 形态成本低，一份提示词加几个脚本就能补上模型能力的缺口。

**已知限制：**
- 10 分钟以上精度下降，长视频必须圈时间段用
- 无字幕视频需配 Whisper key（Groq 或 OpenAI），多一步折腾
- 私有平台一律不支持，yt-dlp 够不到的链接它也够不到

## 实操验证

作者用一条抖音解说《当幸福来敲门》的视频实操测试：
- 抖音在 yt-dlp 支持列表中，链接直接解析下载
- 抖音无外挂字幕轨，转录交给 Whisper
- 结果：回答带画面细节（电影场景、人物动作），不是"这是一个励志视频"的片汤话

## 战略分析

### 与 Hermes 工具链的对照

| 维度 | claude-video | Hermes Agent 当前体系 |
|------|-------------|---------------------|
| 定位 | Claude Code/Codex 的 Skill 插件 | 全功能 Agent 平台（多端交互+技能系统） |
| 视频能力 | yt-dlp 下载 + ffmpeg 抽帧 + Whisper 转录 | [kb-archive-video](https://hermes-agent.nousresearch.com/docs) 已有 Bilibili/YouTube/抖音视频归档管线，含 ASR 转录 |
| 帧级视觉理解 | 将帧作为图片送入 Claude 逐帧读取 | 目前无对应能力——Hermes 的视频归档侧重转录和元数据，不涉及帧级视觉问答 |
| Skill 格式 | 标准 SKILL.md，兼容 50+ Agent 系统 | 自有技能系统（~/.hermes/skills/），格式不兼容但理念相通 |
| 使用模式 | `/watch` 命令交互式问答 | 批处理归档为主，缺乏实时视频问答交互 |

### 能力差距与借鉴点

1. **帧级视觉问答**是 Hermes 当前视频管线的明确缺口。现有流程走到 ASR 转录就停了，视频画面信息（UI 操作、演示动画、PPT 内容）全部丢失。如果用户有分析录屏、拆解教程视频的需求，这个能力就很有价值。

2. **实现思路可以直接复用**：Hermes 已有 yt-dlp + ffmpeg 依赖（在视频转录管线中用到了），Whisper 转录也有（faster-whisper tiny）。缺的只是「抽帧 → 将帧送入多模态模型 → 回答用户问题」这层。理论上可以新建一个 Hermes 技能（如 `watch-video`）来封装这个能力。

3. **架构差异**：claude-video 走的是 Claude Code 的交互式 `/watch` 命令，Hermes 是 WeChat/Telegram/CLI 多端输入。如果要整合，更适合做成 cronjob 或交互式技能，用户在聊天中丢视频链接 + 问题，Hermes 完成抽帧+分析后返回结论。

4. **长视频限制是共性挑战**：30 分钟以上稀疏扫描的问题任何方案都绕不过，聚焦模式（圈时间段）是通用解法。

### 整合可能性

如果用户有经常需要分析视频画面（bug 录屏、教学视频拆解、短视频分析）的需求，值得新建一个 Hermes skill 来包装这个能力。核心组件已齐备（ffmpeg、Whisper、多模态模型访问），主要工作是把「抽帧 → 视觉理解」的管线串起来，适配到 Hermes 的多端交互模式中。

## 归档日志

- 2026-07-21 归档
