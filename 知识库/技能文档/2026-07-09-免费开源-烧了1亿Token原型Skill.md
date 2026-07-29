---
title: "免费开源：烧了1亿Token原型Skill，真的好用"
source: "产品经理老王霸"
source_url: "https://mp.weixin.qq.com/s/pHspNo01LnNi3h04iEnlhg"
date: "2026-07-09"
tags: [Codex, Skill, Figma, 原型, 开源]
---

> 老王霸 AI Lab 开源的 **产品原型生成 Skill** — 一句话需求 → IA推导 → Design Plan → Anti-Slop检查 → page.ui.yaml → Figma 可编辑稿，全程自动化。

GitHub: [pmlaowangba-lab/laowangba-pmprototype-skill](https://github.com/pmlaowangba-lab/laowangba-pmprototype-skill)

## 核心流程

```
一句话需求 → IA分析 → Design Plan → 反AI味检查(7道) → page.ui.yaml → Figma渲染
```

每步有强制检查关卡，不过关不准往下走。使用 `DESIGN.md` 写死设计 Token（颜色、间距、字号、圆角），AI 只能从 token 表取值，防止抽卡。

## 使用方式

1. 任意 Coding Agent（Codex / Cursor / Claude Code），需支持 Skill 加载
2. Figma 账号（免费版即可）
3. Figma MCP 或 Figma 插件配好

安装：把 GitHub 链接丢给 Agent 让它自己装，"原汤化原食"。

使用：`使用产品原型生成Skill，生成一个会员系统前台页面原型`

## 开发复盘关键点

- 用自然语言约束 AI 审美行不通 → 改用硬规则 + 7道强制检查
- 颜色/间距/字号/圆角写死 token 表 → 防止抽卡
- v0.1 版 Figma 执行侧复杂页面偶尔丢 region 结构，需手动 patch

## 归档日志

- 2026-07-16 归档
