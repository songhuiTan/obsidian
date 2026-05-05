---
title: LLM Wiki 模式 (Karpathy)
created: 2026-05-05
updated: 2026-05-05
type: concept
tags: [knowledge-mgmt, architecture, workflow, rag]
sources:
  - raw/articles/hermes-llm-wiki-实战-2026-04-17.md
  - SCHEMA.md
confidence: high
---

# LLM Wiki 模式

## 定义

LLM Wiki 是 Andrej Karpathy 提出的知识管理模式：让 LLM 将原始材料**编译**（compile）为结构化的 Wiki，之后所有查询都发生在这个 Wiki 上，而非每次从原始文档中临时检索。

核心比喻：**Obsidian 是你的 IDE，LLM 是你的程序员，Wiki 就是你的代码库。**

## 与 RAG 的本质区别

| 维度 | 传统 RAG | LLM Wiki |
|------|---------|----------|
| 查询方式 | 每次从零检索 | 查询已编译的知识 |
| 知识沉淀 | 不沉淀 | 每次投喂触发页面更新 |
| 维护负担 | 低（但质量不增长） | 初期高（但复利增长） |
| 幻觉风险 | 较高（每次重新拼） | 较低（结构已知） |
| 最适合量级 | 大规模（>40万字） | 个人知识库规模（<100篇） |

## 三层架构

### 第一层：raw（原始来源层）
LLM **只读不改**的原始材料。文档、文章、笔记、代码。是"事实的唯一源头"。

### 第二层：wiki（编译层）
LLM 按 SCHEMA 规则编译出来的页面网络。包括 entities/、concepts/、comparisons/、queries/，以及 index.md（总目录）和 log.md（操作日志）。页面间通过 `[[wikilink]]` 双向互联。

### 第三层：SCHEMA（规则层）
`SCHEMA.md` 一个文件，定义：结构、命名规范、标签分类、页面创建阈值、更新策略。是用户和 LLM 之间的契约。

## 实施要点

- **raw 层放什么比工具选什么重要** —— 只放自己写过的、消化过的、干过的内容，才能编译出"你的思想地图"
- **红绿灯原则**：
  - 🟢 绿灯（LLM 全权）：摘要生成、索引更新、链接补全、孤儿页检查
  - 🟡 黄灯（共同审核）：矛盾裁决、概念合并、过时内容作废
  - 🔴 红灯（人类专属）：核心事实写入、价值判断、最终签字

## 相关实现

- [[hermes-agent]] — Nous Research 将 LLM Wiki 打包为内置 skill
- 本 Wiki 即按照 LLM Wiki 模式搭建
