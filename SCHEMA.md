# Wiki Schema

## Domain
AI Agent 生态研究——覆盖 OpenClaw、Agent 框架（OpenDeepCrew、DeerFlow 等）、Harness 体系、Skill 技能开发、Agent 工具链、Claude Code 等 AI Coding 实践，以及相关的行业动态与技术趋势。

## Conventions
- 文件命名：小写字母 + 连字符，中英文混合可保留关键中文（如 `openclaw-skill-开发指南.md`）
- 每个 Wiki 页面以 YAML frontmatter 开头（见下方模板）
- 使用 `[[wikilinks]]` 互链，每页至少 2 个出站链接
- 更新页面时同步更新 `updated` 日期
- 新建页面必须加入 `index.md` 对应章节
- 每次操作必须追加到 `log.md`
- **溯源标记：** 当页面综合 3+ 来源时，在段落末尾添加 `^[raw/articles/source-file.md]` 标记具体来源
- 现有 `知识库/` 目录为原有笔记内容，视为原始材料（Layer 1），只读不修改

## Frontmatter
```yaml
---
title: 页面标题
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: entity | concept | comparison | query | summary
tags: [取自下方分类体系]
sources: [raw/articles/source-name.md]
# 可选质量信号：
confidence: high | medium | low        # 结论的支撑强度
contested: true                        # 存在未解决矛盾时设为 true
contradictions: [other-page-slug]      # 存在矛盾的页面
---
```

### raw/ 文件 Frontmatter
```yaml
---
source_url: https://example.com/article
ingested: YYYY-MM-DD
sha256: <正文内容的 sha256 摘要>
---
```

## Tag Taxonomy

### 框架与平台
- openclaw — OpenClaw 生态相关
- harness — Agent Harness/编排引擎（MyHarness、DeerFlow、OpenSpace）
- agent-framework — 其他 Agent 框架（OpenDeepCrew 等）
- claude-code — Claude Code 相关

### 技能与工具
- skill — OpenClaw Skill 开发与使用
- tool — 工具链（OpenCLI、Playwright、Zread 等）
- integration — 集成（飞书、Obsidian、微信公众号等）

### 技术概念
- agent — AI Agent 通用概念
- memory — Agent 记忆系统
- workflow — 工作流 / 自动化
- architecture — 系统架构
- rag — RAG / 检索增强生成
- fine-tuning — 微调 / 模型优化

### 实践与方法
- tutorial — 教程指南
- practice — 实战经验
- review — 复盘总结
- knowledge-mgmt — 知识管理

### 行业与生态
- company — 公司 / 组织
- open-source — 开源项目
- news — 行业动态
- comparison — 对比分析
- trend — 趋势预测

规则：所有页面标签必须来自此分类体系。如需新标签，先在此添加再使用。

## 页面创建阈值
- **创建页面：** 当实体/概念出现在 2+ 来源中，或在一个来源中处于核心地位
- **追加内容：** 当新来源提及已有页面内容时，追加到现有页面
- **不创建：** 简单提及、次要细节、领域外内容
- **拆分：** 页面超过 ~200 行时拆分为子主题并交叉链接
- **归档：** 内容完全被取代时移至 `_archive/`，从 index 中移除

## Entity Pages
一个实体一页。包含：
- 概述 / 是什么
- 关键事实和时间线
- 与其他实体的关系（`[[wikilinks]]`）
- 来源引用

## Concept Pages
一个概念一页。包含：
- 定义 / 解释
- 当前认知状态
- 未解决问题或争论
- 相关概念（`[[wikilinks]]`）

## Comparison Pages
对比分析。包含：
- 比较对象和比较目的
- 比较维度（优先表格形式）
- 结论或综合判断
- 来源

## 更新策略
当新信息与已有内容冲突时：
1. 比较日期——新来源优先于旧来源
2. 如确实矛盾，同时记录两种说法（含日期和来源）
3. 在 frontmatter 中标记矛盾：`contradictions: [page-name]`
4. 在 lint 报告中标记供用户审阅

## 红绿灯原则（人机分工契约）

基于 Karpathy LLM Wiki 模式中的人机协作规范，明确哪些事 AI 可以自主做、哪些需要一起审、哪些必须人来定。

### 🟢 绿灯区 — AI 全权处理（不需确认）
- 摘要生成和正文提炼
- `index.md` 索引更新和页面新增
- `[[wikilinks]]` 补全和交叉引用维护
- 格式调整、frontmatter 补全、标签标准化
- 孤儿页检查和告警
- 页面拆分（>200 行时）和基础归档
- `log.md` 操作记录追加

### 🟡 黄灯区 — AI 标记 + 人类确认
- **矛盾裁决**：新旧信息冲突时，AI 列出双方论点和时间线，由人类决定保留/替换/并存
- **概念合并**：发现两个页面描述同一事物时，AI 提出合并方案，人类确认
- **过时内容作废**：AI 标出 `confidence: low` 或超 90 天未更新的页面，人类决定是否归档
- **新标签添加**：AI 建议新标签和分类位置，人类确认后加入分类体系
- **queries/ 归档审批**：AI 草拟归档内容，人类确认真实性和价值后再入库

### 🔴 红灯区 — 人类专属（AI 绝不触碰）
- **核心事实的首次写入**：关键数字、决策理由、个人判断—必须人类第一手输入
- **价值判断和立场声明**：Wiki 是你的思想地图，不是世界的折中版
- **最终签字确认**：每页 `confidence: high` 的标记，需人类在 lint 时确认
- **raw/ 层的修改**：原始材料不可篡改，即使发现错误也只能在编译层修正

### 违规后果
- 黄灯/红灯事项如果 AI 擅自处理，可能导致 Wiki 变成"通用知识的二手转述"，失去核心价值
- 每周 lint 时会扫描红绿灯合规情况：标记所有 AI 单方面写入的黄灯/红灯内容，列出来让你复核

## 知识库映射
现有 `知识库/` 目录内容作为 Layer 1 原始材料，Wiki 的 entities/concepts 页面将引用这些内容但不修改。从 `知识库/` 提取实体和概念时，在 `sources:` 中注明路径。
