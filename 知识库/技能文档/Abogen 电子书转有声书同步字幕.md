---
title: Abogen — 电子书转有声书 + 同步字幕
source: https://mp.weixin.qq.com/s/O02XieSaMvCYW4TOoqAn6w
author: 丛林
platform: 微信公众号 · 极客之家
date: 2026-06-18
tags:
  - TTS
  - 有声书
  - 开源工具
  - Kokoro-82M
  - 字幕生成
  - 技能文档
status: archived
---

# Abogen — 电子书拖进去，秒变有声书 + 同步字幕

> GitHub 4.8k Star，MIT 协议。底层 Kokoro-82M，支持 EPUB/PDF/TXT/Markdown/字幕文件转有声书 + 同步字幕。

**GitHub:** https://github.com/denizsafak/abogen

---

## 核心功能

### 1. 文件转音频
桌面端 PyQt6 编写，拖拽 ePub/PDF 进输入框，选语速（0.1~2.0）、声音、输出格式，点 Start 几秒出音频。

性能参考：RTX 2060 Mobile 跑 3000 字，**11 秒出 3 分多钟音频**。CPU 也能跑。

输出格式：WAV、FLAC、MP3、**OPUS**（最实用，压缩狠体积小）、**M4B**（苹果有声书格式，带章节信息）。

### 2. 同步字幕生成
字幕粒度可调：按行切、按句子切、按句子加逗号切、按单词数切（1 词/2 词等）。**句子高亮模式**，读到哪里亮到哪里。

- 英文：Kokoro 输出 token 级时间戳，支持单词级字幕（需开 spaCy 处理 "Mr." "Dr." 等缩写）
- 中文：走句子和逗号拆分，字幕稍长但体验没差

### 3. 语音混音器
不同 Kokoro 语音模型按权重混合，捏出自定义声音，存成 profile 下次复用。

支持语言：美式英语、英式英语、西班牙语、法语、印地语、意大利语、日语、巴西葡萄牙语、**中文普通话**（中日文需额外装 misaki 包）。男女声都有。

### 4. 队列模式
批量处理文件，每个文件绑独立配置，也可开 Override 统一覆盖。悬停查看每个文件的参数。

### 5. Web UI
`abogen-web` → `localhost:8808`（Flask）。比桌面端多三项：
- **Supertonic TTS** — 额外 TTS 引擎
- **LLM 文本规范化** — 处理 don't/can't/I'll 等缩写，支持 Ollama 本地或 OpenAI API
- **Audiobookshelf 直连** — 生成完直接推入库

### 6. 章节标记与元数据
- 自动插入 `<<CHAPTER_MARKER:章节标题>>`，按章节拆独立音频文件，单章出错只重跑那章
- M4B 支持元数据标签：`<<METADATA_TITLE:标题>>`、`<<METADATA_ARTIST:作者>>` 等
- 封面图自动从 ePub/PDF 提取嵌入
- 支持时间戳文本：txt 中写 `HH:MM:SS` 格式时间码，按时间轴出音频（配音/定时旁白）

## 安装

```bash
# Web UI
abogen-web
# 桌面端
abogen
```

- **Windows:** 直接跑 `WINDOWS_INSTALL.bat`
- **Mac:** 先装 `espeak-ng`，再 `uv tool install`，Apple Silicon 注意 Kokoro MPS 支持
- **Linux:** NVIDIA 直装，AMD 走 ROCm，服务器看 Docker Compose

## 不足
- 桌面端与 Web 端功能未对齐
- 单词级字幕中文用不了（Kokoro 限制）
- 两端 UI 风格不统一

## 与现有工具的关联

这个工具链跟我们的 B 站/抖音视频归档中的 ASR（faster-whisper）管线可以互补——ASR 做语音转文字，Abogen 做文字转语音 + 字幕。如果后续需要批量生产配音内容或本地有声书，值得评估 Kokoro-82M 在 Hermes 中的集成可能。
