---
title: CoreCoder
created: 2026-07-21
updated: 2026-07-21
type: entity
tags: [tool, claude-code, harness, methodology]
sources:
  - 技能文档/2026-07-02-CoreCoder-Claude-Code底层开源复刻.md
confidence: medium
---

# CoreCoder

## 概述

CoreCoder（GitHub: he-yufeng/CoreCoder）是一个轻量级开源编码 Agent，旨在让开发者理解 [[Claude Code]] 的核心架构，而非替代。核心数据：仅 **1,714 行物理代码（18 个文件）**，其中 engine 核心仅 1,081 行。它是目前最精简的 Claude Code 架构教育级复刻实现。

## 核心能力

保留了 Claude Code 最关键的底层机制：
- **主循环**（agent.py）—— Agent 运行的主事件循环
- **模型接口**（llm.py）—— 模型怎么要工具
- **上下文管理**（context.py）—— 上下文不爆的核心机制
- **工具系统**（tools/bash.py 等）—— 工具定义与执行
- **Session 管理** —— 会话生命周期
- **并行工具执行** —— 多工具同时调用
- **三层 context 压缩** —— 防止上下文爆炸
- **bash 危险命令拦截**（regex blacklist）—— 基础安全防护
- **文件读写、shell 执行、sub-agent 调度** —— 基本 Agent 能力

## 配置与兼容性

- 默认走 OpenAI-compatible API
- DeepSeek/Ollama 等换 `OPENAI_BASE_URL` + 模型名即可
- 不兼容的通过 LiteLLM 接入 100+ 服务商

## 学习建议

1. 从 `agent.py`（主循环）开始
2. 接着看 `llm.py`（模型怎么要工具）、`context.py`（上下文不爆）、`tools/bash.py`
3. 先不加 MCP、RAG 等外围模块
4. 核心看：模型怎么要工具 -> 工具怎么回填 -> 上下文怎么不爆

## 关键事实与日期

- 2026-07-01：CoreCoder 项目发布
- 2026-07-02：知识库归档
- 1,714 行物理代码，18 个文件
- engine 核心仅 1,081 行
- 危险命令拦截基于 regex blacklist，非安全沙箱
- 接不可信输入仍需容器隔离+权限控制

## 相关文档

- [[Claude Code]] —— CoreCoder 复刻的原型，闭源 50 万行+代码
- [[OpenDev]] —— Rust 实现的生产级开源 Claude Code 替代，81 页论文架构全拆解
- [[Trellis]] —— 团队级 Agent Harness，CoreCoder 可作为教育级入口理解 Harness 概念
- [[WorkBuddy]] —— 生产级方法论，CoreCoder 体现了"能用工具"和"能做任务"两个阶段
- [[Hermes Agent]] —— 同为 Agent 框架，不同复杂度层级的参考

## 与其他实体的关系

CoreCoder 是 [[Claude Code]] 体系中最适合入门学习的技术实现。与 [[OpenDev]]（Rust 生产级实现，81 页论文）相比，CoreCoder 以极简代码量（1,714 行 vs OpenDev 的完整工程）呈现了 Agent 编码工具的核心骨架。CoreCoder 的"三层 context 压缩"和"并行工具执行"设计，体现了 [[WorkBuddy]] 六阶段中"能做任务"和"能跑长任务"的底层工程机制。

## 来源引用

- 技能文档/2026-07-02-CoreCoder-Claude-Code底层开源复刻.md
