---
title: Skill 体系
created: 2026-07-21
updated: 2026-07-21
type: concept
tags: [skill-system, agent-framework, harness, methodology]
confidence: medium
sources:
  - AI-Agent-Skill-工程化01 (06-01)
  - 歸藏Skills万字长文 (06-11)
  - OpenSpace-Skill进化 (07-09)
  - 腾讯SkillHone (07-03)
  - 阿里SkillWeaver (07-06)
  - Resource2Skill (07-03)
  - ANYTHING2SKILL (06-11)
  - OpenSquilla自进化 (06-11)
  - SkillDAG技能关系图 (06-04)
  - Skill Quality Gate (07-02)
  - yao-meta-skill (06-20)
---

# Skill 体系

Skill 体系是 AI Agent 能力可复用的工程单元架构。每个 Skill 封装了特定领域的能力——包括 prompt 模板、工具调用链、上下文管理逻辑和输入输出规范——使得 Agent 的能力可以被标准化定义、版本管理、组合复用。Skill 是 [[harness-engineering]] 的执行原子，也是 [[loop-engineering]] 中循环步骤的基本单位。

## 核心能力

- **标准化封装**：将 Agent 的特定能力封装为可复用的模块，包含输入 schema、执行逻辑、输出规范和质量标准。
- **动态组合**：通过 [[multi-agent-orchestration]] 将多个 Skill 组合成复杂工作流，Skill 之间通过 DAG 关系图（SkillDAG）编排。
- **自进化**：Skill 可以在运行时根据执行结果自我优化，OpenSpace 和 OpenSquilla 展示了 Skill 自动进化的可行性。
- **质量门禁**：Skill Quality Gate 定义了 Skill 发布的验收标准，确保进入生产环境的 Skill 是可靠的。

## 关键实现

| 项目 | 特点 |
|------|------|
| **腾讯 SkillHone** | Agent 技能进化与记忆持久化，解决 Agent 失忆问题 |
| **阿里 SkillWeaver** | 跳过加载全部工具，Token 消耗直降 99% |
| **歸藏 Skills** | 万字长文系统阐述 Skill 工程化方法论 |
| **Resource2Skill** | 将任何资源自动转化为 Skill |
| **ANYTHING2SKILL** | 更加通用的资源到 Skill 的转换框架 |
| **yao-meta-skill** | 元 Skill 概念，用于生成和管理其他 Skill |

## 与相关概念的关系

| 概念 | 关系 |
|------|------|
| [[harness-engineering]] | Harness 是 Skill 的运行环境，Skill 是 Harness 的执行单元 |
| [[loop-engineering]] | 循环中的每个步骤可映射为一个 Skill 调用 |
| [[agent-evaluation]] | Skill 质量评测是 Agent 评测体系的核心组成部分 |
| [[agent-governance]] | Skill 的权限控制和调用审计属于治理范畴 |
