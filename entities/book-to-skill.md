---
title: book-to-skill
created: 2026-06-04
updated: 2026-06-04
type: entity
tags: [tool, skill, knowledge-mgmt, open-source]
sources: [知识库/技能文档/2026-06-04-一个开源的Skill阅读神器book-to-skill.md]
confidence: high
---

# book-to-skill

book-to-skill 是一个开源工具，将 PDF、EPUB、DOCX、Markdown 等格式的书籍或文档资料打包成 [[claude-code]] 可调用的 Skill 文件。解决"读完书就忘、AI 幻觉、上下文撑爆"三大痛点。

## 工作原理

1. **格式判断**：技术类书籍 → Docling 线（保留表格 + 代码块）；叙事类书籍 → 纯文本提取
2. **深度分析**：提取核心框架和章节索引，生成 `SKILL.md`
3. **章节拆分**：每章独立文件，配有术语表、模式表和速查表
4. **按需加载**：只有问到某一章才加载对应内容，不提前灌满上下文

## 与传统方式对比

| 方式 | 问题 |
|------|------|
| 直接上传 PDF | 每次对话重新烧 Token，幻觉极高 |
| RAG 检索 | 拼答案看运气，不保证准确 |
| **book-to-skill** | AI 调用已消化好的框架，准确率更高 |

## 安装

```
请读取下面链接文件内容，并安装好这个 Skill：
https://raw.githubusercontent.com/virgiliojr94/book-to-skill/master/SKILL.md
```

## 使用

```
/book-to-skill ~/path/to/your-book.pdf
```

## 相关页面

- [[claude-code]] — 主要在 Claude Code 中使用
- [[skill-development]] — Skill 开发方法论
- [[knowledge-management]] — 知识管理实践
