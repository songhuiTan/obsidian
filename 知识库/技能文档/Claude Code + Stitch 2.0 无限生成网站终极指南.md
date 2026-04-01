---
title: Claude Code + 全新 Stitch 2.0 才是无限生成 80,000 元级别网站终极指南
source: https://mp.weixin.qq.com/s/YybUFN6krThrtP-c0_XDug
author: 请叫我"一点儿"
archived: 2026-03-23
tags: [AI, claude-code, stitch, 网站开发, MCP, 设计系统, agent]
---

# Claude Code + 全新 Stitch 2.0 才是无限生成 80,000 元级别网站终极指南

> Jack Roberts 把 Google Stitch（Gemini 3.1 驱动的设计代理）接进 Claude Code，再用 anti-gravity 把流程"串"成一条流水线。

## 核心痛点：一致性

很多人卡在建站，不是卡在代码，而是卡在**一致性**：
- 首页很惊艳，第二页开始崩
- 颜色、字体、间距、按钮风格像"不同人做的"
- 你想扩到 10 个页面，结果每个页面都要重新发明轮子

Stitch 的核心思路：**从视觉出发，你看得见、改得动、还能把"美"复用成体系。**

## Stitch 关键功能

### 1. Ideation（构思）—— 先调研再给三套路子

丢需求进去，它会：
- 调研竞品
- 给出**三套可落地的设计方向**（极简、强调实用、Stripe/Notion 启发式）
- 把调研结果变成可用的设计要素——字体、结构、模块、交互建议
- ROI 计算器、FAQ、评价墙等"高转化组件"作为默认配置

**价值**：把你从'不知道要什么'的混沌里拉出来。

### 2. Redesign（重设计）—— 先用图把目标定死

真正让首次输出质量暴涨的，不是更会写提示词，而是——**先用图把目标定死**。

操作：
1. 找一个觉得很酷的网站（或 Dribbble 截图）
2. 丢进 Stitch 的 Redesign
3. 提需求：顶部主视觉、评价、合作公司、工作原理、FAQ、CTA 等
4. Stitch 用 Nano Banana 2 吐出一个**完整概念设计**

编辑方式像"搭积木"：点中元素就能改，文字、模块、比例都能直接调。

> 因为"美"很难靠文字一次讲清，但一张概念图能把光影、留白、氛围、主次关系全部锁定。

**限制**：Redesign 每天约 15 次点数。

### 3. Design MD —— 把风格"编码化"

把满意页面拆成可复用的设计蓝图：
- 列出所有 screen
- 抓取 HTML
- 抽取设计 Token（颜色、排版、间距、组件风格、氛围关键词）
- 输出项目级元数据 + 可复用规范

**商业价值**：
- 后续加页面不会"换设计师"
- 营销页、定价页、文档页风格统一
- 同一风格可卖给不同细分行业
- A/B 测试多个页面仍保持品牌一致

### 4. Loop + Enhanced Prompt —— 批量生成

- **Stitch Loop**：自治多页生成，Claude 做完一页接力给自己
- **Enhanced Prompt**：把模糊需求变成专业可控指令
- **Shadcn UI**：把设计桥接为符合最佳实践的组件
- **Remotion**：用 Stitch 屏幕做带缩放的演示走查视频

## MCP 接入方式

在 anti-gravity 里：MCP service 搜索 Stitch → install → 粘贴 API Key → refresh

在 Claude Code 里：用命令把 MCP server 加进去 → 填 API Key

## 完整流水线

```
调研/构思 → 视觉生成/重设计 → 抽取 Design Token → 批量生成多页 → 组件化 → 部署上线
```

最后可以直接发布到 GitHub 和 Vercel。

## 视频来源

https://www.youtube.com/watch?v=1aI7pAlkz4w
