---
title: 扣子（Coze）
created: 2026-06-04
updated: 2026-06-04
type: entity
tags: [agent-framework, open-source, agent, workflow, multi-agent]
sources: [知识库/技能文档/2026-06-04-用扣子搭建Agent团队【数字游牧人懒人包】.md]
confidence: high
---

# 扣子（Coze）

扣子是字节跳动推出的 AI Agent 搭建平台，支持多 Agent 协作、Codex 集成、GPT-Image2、Seedance 2.0 视频生成等多种能力。用户可以在一个项目中组合不同能力的 Agent，形成协作团队。

## 核心能力

- **多 Agent 协作**：不同角色 Agent（产品、设计、视频、开发）在同一项目内协作，共享上下文
- **Codex 集成**：调用 Codex 执行代码生成、图像生成（GPT-Image2）
- **多媒体生成**：结合 Seedance 2.0 生成视频
- **人机协作**：真人也可拉入项目，复用 Agent 产出的中间成果
- **上下文共享**：前面 Agent 的产出可直接被后面 Agent 使用，无需重复解释

## 工作流示例

1. 产品经理 Agent 产出需求文档
2. 美工 Agent 基于需求生成产品说明图
3. 导演 Agent 基于分镜图生成宣传视频
4. 开发 Agent 基于素材开发网站
5. 真人同事加入项目，进一步迭代

## 与其他平台关系

- [[openclaw]] — 开源自托管方案 vs 扣子云平台方案
- [[opendeepcrew]] — 同为多 Agent 协作，架构不同
- [[claude-code]] — 扣子内部通过 Codex 调用类似能力

## 评价

扣子的优势在于**降低多 Agent 协作的门槛**：无需部署、无需编排代码，通过项目空间即可实现 Agent 团队协作。适合快速原型验证和中小企业场景。
