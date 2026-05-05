---
title: SOUL.md — AI Agent 人格定义文件
created: 2026-05-05
updated: 2026-05-05
type: concept
tags: [configuration, customization, workflow, tutorial]
sources:
  - raw/articles/hermes-full-config-guide.md
confidence: high
---

# SOUL.md — AI Agent 人格定义文件

## 定义

SOUL.md 是 Hermes Agent 的"灵魂文件"，存放于 `~/.hermes/SOUL.md`，用于定义 AI 的人格特质、工作风格、专业领域和用户身份。让 AI 从通用助手变成个性化的"数字员工"。

## 核心维度

| 维度 | 说明 | 示例 |
|------|------|------|
| 人格特质 | 严谨工程师 / 创意营销 / 耐心导师 | "我是一名全栈工程师" |
| 沟通风格 | 简洁直接 / 详细解释 / 幽默风趣 | "简洁直接，避免废话" |
| 专业领域 | 编程/设计/金融/营销 | "Web开发、AI应用、自动化流程" |
| 用户画像 | 让 AI 知道"你是谁" | "一人公司创始人，主攻技术出海" |

## 快速方案

使用现成的角色模板仓库，包含 211 个中文角色模板、18 个部门分类：
- GitHub：https://github.com/jnMetaCode/agency-agents-zh

## 示例结构

```markdown
# 核心人格
你是谁：我是一名全栈工程师，同时也负责产品设计和市场营销。
沟通风格：简洁直接，避免废话；提供可执行的方案，而不是理论。
专业领域：Web开发、AI应用、自动化流程、内容创作。

# 工作偏好
代码风格：遵循最佳实践，注重可读性和可维护性。
文档要求：提供完整注释，输出Markdown格式。
时间管理：优先处理紧急且重要的任务，提供明确的时间估算。

# 长期目标
短期：完成当前项目的MVP版本。
长期：建立个人品牌，实现"一人公司"模式。

# 禁忌
- 不要过度道歉或客套
- 不要提供没有验证的信息
- 不要忽略安全风险
```

## 相关实体

- [[hermes-agent]] — SOUL.md 是 Hermes 个性化配置的核心
- [[hindsight]] — 与 SOUL.md 配合，人格 + 记忆构成完整 Agent 个性化
