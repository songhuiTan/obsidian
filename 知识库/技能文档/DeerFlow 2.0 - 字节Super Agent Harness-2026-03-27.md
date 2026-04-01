---
title: DeerFlow 2.0 - 字节开源 Super Agent Harness
source: https://mp.weixin.qq.com/s/xhs6yJtT6ZGEDdijg8D0fw
author: 智猩猩AI / 程明月/黄超/付宇轩
archived: 2026-03-27
tags: [AI-Agent, Harness, 开源, 字节跳动, LangGraph, DeerFlow]
---

# DeerFlow 2.0 - 字节开源 Super Agent Harness

GitHub 47.3k ⭐ | https://github.com/bytedance/deer-flow

## 什么是 Agent Harness

Agent Harness 是包裹在 AI 模型周围的基础设施，专门用于管理长期任务。位于 Agent Framework 之上的更高层架构。提供预设 Prompt、工具调用标准化、生命周期钩子、开箱即用的规划/文件系统/子智能体管理能力。

> "Agent 是强壮却野性难驯的骏马，Harness 不提供动力，却能牢牢牵住方向、稳住步伐。"
> — Philipp Schmid (前 Hugging Face 工程师)

## AI 工程三次跃迁

1. **2023-2024 提示工程** — 教人类怎么跟 AI 说话
2. **2025 上下文工程** — 精算给 AI 看什么信息
3. **2026 Harness** — 为模型构建可执行、可信赖、可长期运转的数字世界

## DeerFlow 2.0 核心能力

### 1. Skills 与 Tools
- 内置：研究、报告生成、幻灯片、网页生成、图文视频创作
- 一键添加/替换/组合 skills，按需渐进加载，不浪费 token
- Tools 可插拔：网页搜索/抓取、文件操作、Bash 执行
- 通过 MCP Server 或 Python 函数无限扩展

### 2. Sub-Agents（子智能体）
- 复杂任务自动拆解
- 主智能体按需动态拉起子智能体
- 每个子智能体：独立上下文、工具、终止条件
- 可并行执行，主智能体汇总结果
- **处理从几分钟到几小时的不同复杂度任务**

### 3. Sandbox 与文件系统
- 每个任务在隔离 Docker 容器运行
- 完整文件系统：skills / workspace / uploads / outputs
- agent 可读写文件、执行 bash 命令和代码、查看图片
- 可审计、会隔离、session 之间不互相污染

### 4. Context Engineering
- 子智能体上下文完全隔离
- 自动总结、压缩、持久化中间结果
- 超长任务不爆上下文

### 5. 长期记忆
- 跨会话积累个人偏好、写作风格、技术栈
- memory 保存在本地，控制权在用户手里

### 6. 其他特性
- Claude Code 直接交互（`/claude-to-deerflow`）
- 内嵌 Python Client
- 多模型兼容
- 集成智能搜索抓取工具 InfoQuest

## 快速开始

```bash
git clone https://github.com/bytedance/deer-flow.git
cd deer-flow
make config           # 生成配置
# 编辑 config.yaml 配置模型，编辑 .env 设置 API key
make docker-init      # 拉取 sandbox 镜像
make docker-start     # 启动服务
# 访问 http://localhost:2026
```

### Claude Code 集成

```bash
npx skills add https://github.com/bytedance/deer-flow --skill claude-to-deerflow
# 在 Claude Code 中使用 /claude-to-deerflow 命令
```

## 技术栈

- LangGraph + LangChain
- Docker（sandbox）
- 多模型兼容（OpenAI / OpenRouter 等）
- MCP Server 扩展

## 与 OpenClaw 的关联

文章底部提到 2026 中国生成式 AI 大会将设 OpenClaw 技术研讨会。DeerFlow 的 sub-agent + memory + sandbox 架构与 OpenClaw 的 sessions_spawn + memory 系统有相似的设计理念。
