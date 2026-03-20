---
title: 装了一个Skill，PPT从灾难变成了咨询公司水准
author: 大刘
date: 2026-03-20
source: https://mp.weixin.qq.com/s/hIZIuHkCvsN562DubP8C8A
tags:
  - PPT
  - Skywork
  - OpenClaw
  - Skill
  - Prompt
  - 办公自动化
---

# 装了一个Skill，PPT从灾难变成了咨询公司水准

> 作者：大刘
> 天工（Skywork）PPT Skill 使用教程

## 核心观点

**Skill不是外挂，是给AI塞了一本设计手册。** 同一个AI，有没有人教它规矩，活儿完全不是一回事。

裸模型做PPT就像让没学过设计的实习生徒手画——能力有，审美没有。

## 使用环境：SkyClaw

SkyClaw 是天工做的云端 AI Agent（OpenClaw）：
- 地址：https://skywork.ai/home_agent/skyclaw
- 免费体验 5 小时
- 预装全套 Skills：PPT、Design、Excel、Document、Search
- 可视化界面，能看到 Agent 操作过程
- 输出 .pptx 源文件，可用 PowerPoint/Keynote/WPS 编辑

## PPT Prompt 万能模板

```
帮我做一份关于[主题]的PPT，[N]页。

风格参考[McKinsey咨询报告/苹果极简风/科技发布会风]，配色使用[深蓝+白+浅灰]。

请先用Deep Research搜索最新的[行业]数据，引用具体数字和来源。

每页文字不超过[5]个要点，标题[28]号，正文[16]号，
每页必须有一张配图或图表，留白不少于30%。
```

## 三个关键要素

### 1. 风格方向
- 不指定风格 → 配色随机，千篇一律
- 加一句风格参考 → 打开模型学过的千万份真实PPT设计经验
- 示例：McKinsey咨询报告、苹果极简风、科技发布会风、日系杂志风

### 2. 真实数据
- 不加搜索指令 → 模型会编数据
- 加 Deep Research → 扒论文、报告、行业数据，引用真实来源

### 3. 排版约束
- 不限制 → 模型每页写800字小作文
- 加约束 → 自动切换图示优先模式（图标代替段落、时间轴代替列表）

## Skywork 全套办公 Skills

| Skill | 用途 |
|-------|------|
| PPT Skill | 制作高质量演示文稿 |
| Design Skill | 生成配图 |
| Excel Skill | 数据图表、自动化抓取 |
| Document Skill | 报告撰写、竞品追踪 |
| Search Skill | 研究性搜索 |

### 开源地址
- PPT: https://clawhub.ai/gxcun17/skywork-ppt
- Design: https://clawhub.ai/gxcun17/skywork-design
- Excel: https://clawhub.ai/gxcun17/skywork-excel
- Document: https://clawhub.ai/gxcun17/skywork-document
- Search: https://clawhub.ai/gxcun17/skywork-search

## 定价

- **免费**：新用户 5 小时体验，开源 Skill 免费使用
- **Ultra 会员**：$249.99/月，送 $1,250 高端模型 token 额度（Opus 4.6、GPT-5.4、Gemini 3.1 Pro）

## 核心洞察

> Prompt不是咒语，是你给AI的设计brief。风格、数据、排版——想清楚这三件事再按发送，比跑十次碰运气强一百倍。

> Skill的本质，不是让AI更强。是让你能更清楚地告诉它，你要什么。

> 你做不出好PPT，从来不是因为不会设计。是因为你还没想清楚你要说什么。
