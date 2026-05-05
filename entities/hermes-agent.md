---
title: Hermes Agent
created: 2026-05-05
updated: 2026-05-05
type: entity
tags: [agent-framework, open-source, tool, cli]
sources:
  - raw/articles/hermes-llm-wiki-实战-2026-04-17.md
  - raw/articles/hermes-feishu-resource-library.md
  - raw/articles/hermes-full-config-guide.md
confidence: high
---

# Hermes Agent

## 概述

Hermes Agent 是 Nous Research 在 2026 年推出的开源自改进 AI Agent（MIT 协议），GitHub 已超 6 万星标。核心亮点是内置**闭环学习系统**（Closed Learning Loop）：它能从经验中自动创建技能、持续优化、跨会话持久记忆，并构建对用户的深度模型。

不是简单的聊天机器人或 IDE 插件，而是一个真正能"越用越聪明"的持久化自治代理。

## 核心机制

### 1. 技能自动生成与自改进
完成一个复杂任务后，自动将整个思考过程、工具调用序列和最终方案提炼成结构化技能文件（`~/.hermes/skills/`）。下次遇到类似需求，优先复用并迭代。

### 2. 三层记忆系统
- **短期**：当前会话上下文
- **中期**：SQLite 全文搜索 + 向量检索
- **长期**：Honcho 用户建模（跨会话构建偏好画像）

### 3. 用户模型与个性化
通过 `SOUL.md` 全局人格文件和项目级上下文文件深度定制。越用越懂你的写作风格、代码偏好、技术栈。

## 能力全景

| 维度 | 详情 |
|------|------|
| 模型兼容 | OpenRouter(200+)、Ollama、本地 vLLM、Anthropic、OpenAI 等 |
| 内置工具 | 40+：网页搜索、浏览器、视觉、TTS、终端、文件、代码解释器 |
| 平台支持 | CLI、Telegram、Discord、Slack、WhatsApp、Signal、Email 等 15+ |
| 部署方式 | 本地、Docker、SSH、VPS、Modal/Daytona serverless |
| 扩展协议 | MCP 服务器、ACP 编辑器集成 |

## 核心配置项（满配 vs 裸装）

| 模块 | 裸装 | 满配方案 |
|------|------|---------|
| 人格定义 | 无 | `SOUL.md` 定义人格特质、工作风格、专业领域 |
| 长期记忆 | MEMORY.md（~2200字符上限） | **Hindsight** 自动提取、知识图谱、云端无上限 |
| 网页抓取 | 无 | Jina Reader + Crawl4AI + Scraping + Camoufox |
| 搜索能力 | 无 | Tavily（1000次/月免费）+ DuckDuckGo（无限免费） |
| 文档处理 | 仅支持纯文本 | Pandoc（万能格式转换）+ Marker（PDF高精度） |
| 语音 | 无 | Whisper（语音识别）+ Edge TTS（语音合成） |
| 图片生成 | 无 | Fal.ai + FLUX Skill |
| Token管控 | 无 | RTK（压缩60-90%）+ Tokscale（实时监控）+ hermes-hudui（Web UI） |

## 生态资源

- **awesome-hermes-agent** — 一站式教程/工具/Skill/案例汇总
- **hermes-ecosystem** — 80+ 工具可视化地图
- **wondelai 380 跨平台 Skill** — 一次性覆盖各种场景
- **awesome-agent-skills** — 1000+ 技能库按需挑选

