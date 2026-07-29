# Wiki Schema

## Domain
AI Agent 学习、实践与归档知识库。涵盖 Agent 框架/Harness 工程、Claude Code & Codex、RAG 与知识检索、知识管理与归档、工具与实用技能、企业级 AI & AIOps、OpenClaw 生态、AIGC 与内容创作。

## Architecture: Option B — 集成到现有 Vault

本 wiki 是 llm-wiki Option B 实现，集成到已有的 Obsidian 知识库 Vault。

### 分层结构

**Layer 1 — 用户自有内容（不可变，只读）：**
以下目录为用户自有内容，Agent 读取但不修改（除非用户明确要求整理）：
- `技能文档/` — 归档文章、视频转录、技能文档（182 篇文件，含子目录 hermes-llm-wiki实战 和 开源工具）
- `教程指南/` — 完整教程与指南（5 篇）
- `复盘日记/` — 学习复盘与踩坑记录（2 篇）
- `索引目录/` — 索引文件（1 篇）
- `InStreet经验/` — InStreet 平台经验（3 篇）
- `ClawBot技能学习/` — ClawBot 技能（1 篇）
- `OpenClaw系统/` — OpenClaw 系统配置（1 篇）
- `assets/` — 图片、封面等资源
- `README.md` — Vault 总览
- `快速导航.md` — 分类导航索引

**Layer 2 — Wiki 自有内容（Agent 管理）：**
- `entities/` — 核心实体页（工具、框架、平台、人物）
- `concepts/` — 核心概念页（架构模式、方法论）
- `comparisons/` — 对比分析页
- `queries/` — 有价值的查询结果归档
- `_archive/` — 已归档的 wiki 页
- `SCHEMA.md` — 本文件
- `log.md` — 动作日志
- `index.md` — Wiki 内容索引

### 引用方式
Wiki 页通过 `sources:` frontmatter 引用用户内容，例如：
```yaml
sources:
  - 技能文档/2026-06-20-从原始到Agentic-8种RAG架构深度解析与生产实践指南.md
  - 技能文档/kb-builder — 一键搭建AI知识库（Claude Code Skill + ChromaDB + MCP）.md
```

## Conventions

### 文件命名
- 用户内容文件：使用现有命名约定（日期前缀 + 中文描述：`YYYY-MM-DD-标题.md`）
- Wiki 页：全小写+连字符（`tool-name.md`）
- 中文术语必要时保留：`rag-mcp-知识库.md`

### 导航体系
- `README.md` — Vault 总览入口（数量统计、目录概览、归档流程）
- `快速导航.md` — 所有文章的分类导航索引（按主题分组，含近日更新标记）
- `索引目录/OpenClaw 学习索引.md` — 历史遗留的 OpenClaw 专题索引
- `index.md` — Wiki 内容索引（Layer 2 内容）
- `SCHEMA.md` — 本 schema 文件
- `log.md` — 动作日志

### 更新原则
- 每次修改后更新 README.md 中的计数和日期
- 每次新增文章必须添加到 快速导航.md 对应分类
- 每次操作必须追加到 log.md
- 快速导航末尾日期与 README 总览日期同步

## Frontmatter（Wiki 页）

所有 Wiki 页（Layer 2）必须包含 YAML frontmatter：

```yaml
---
title: 页面标题
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: entity | concept | comparison | query
tags: [from taxonomy]
sources: [技能文档/source-file.md]
confidence: high | medium | low
---
```

## Tag Taxonomy

- **框架与工程**: agent-framework, harness, skill-system, orchestration, evaluation
- **工具**: claude-code, codex, openclaw, hermes, mcp, tool
- **知识管理**: rag, knowledge-base, wiki, archive, memory
- **企业**: enterprise, aiops, production, integration
- **内容**: aigc, ppt, video, design, writing
- **方法**: methodology, tutorial, comparison, review, case-study

## Page Thresholds

- **创建实体页**：当某个工具/框架在 3+ 篇不同文章中出现且是你关注方向
- **创建概念页**：当某个架构模式/方法论在 2+ 篇分析中出现
- **创建对比页**：当两个同类工具/方法各有优劣、值得并排比较
- **跳过**：随口提及、边缘细节、不在你关注范围内
- **拆分**：超过 200 行时拆分为子主题页

## 红绿灯原则（人类-AI 分工）

**绿灯（Agent 自主执行）**：
- 快速导航的补充和更新
- README 计数的同步
- log.md 记录
- index.md 更新
- 格式标准化

**黄灯（Agent 标记，人类确认）**：
- 内容矛盾处理
- 页面合并建议
- 新标签添加

**红灯（仅限人类）**：
- 核心事实的首次录入
- 价值判断和立场声明
- confidence: high 的最终确认
- raw/ 层（用户内容）的任何修改（除非用户明确要求）
