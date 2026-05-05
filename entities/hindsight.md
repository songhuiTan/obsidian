---
title: Hindsight
created: 2026-05-05
updated: 2026-05-05
type: entity
tags: [tool, memory, cloud, integration]
sources:
  - raw/articles/hermes-full-config-guide.md
confidence: high
---

# Hindsight

## 概述

Hindsight 是一个专为 AI Agent 设计的云端记忆管理系统，作为 Hermes Agent 内置 MEMORY.md 的替代方案。核心能力：自动从对话中提取实体、事实、关系和时间戳，构建知识图谱，实现几乎无上限的长期记忆。

## 对比内置记忆

| 维度 | Hermes 内置 MEMORY.md | Hindsight |
|------|----------------------|-----------|
| 存储上限 | ~2,200 字符 | 云端几乎无限（免费版100MB） |
| 写入机制 | 仅在"觉得重要"时写入 | 自动提取每轮对话 |
| 检索方式 | 全文注入上下文 | 知识图谱 + 语义检索 |
| 结构 | 平铺文本 | 实体-关系-事实结构化 |
| 注入策略 | 全部注入 | 仅注入相关记忆 |

## 功能特性

- **自动提取**：从每轮对话中提取实体、事实、关系、时间戳
- **知识图谱**：结构化存储，建立实体之间的关系网络
- **自动注入**：新会话时，自动注入最相关的记忆到上下文
- **无需手动管理**：对比 MEMORY.md 的手动写入，Hindsight 全自动

## 配置方式

```bash
# 1. 选择 Hindsight 作为记忆后端
hermes memory setup
# → 选择 "Hindsight"

# 2. 获取 API Key
# 访问 https://ui.hindsight.vectorize.io/connect

# 3. 配置到 Hermes
# 编辑 ~/.hermes/config.json → 添加 hindsight_api_key

# 4. 验证
hermes memory status
# ✅ auto-recall: enabled
# ✅ auto-retain: enabled
```

## 局限性

- 免费版 100MB 存储上限，超出需付费
- 依赖网络连接 Hindsight 服务器
- API Key 需妥善保管，不应上传公开平台

## 相关实体

- [[hermes-agent]] — 通过 Hindsight 增强长期记忆能力
