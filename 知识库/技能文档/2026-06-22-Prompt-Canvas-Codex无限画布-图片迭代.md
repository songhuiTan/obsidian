---
source: 微信公众号
author: 林月半子聊AI
url: https://mp.weixin.qq.com/s/bnbeOWjlG2_znoG-2p9Zdw
date: 2026-06-22
tags: [Codex, Prompt Canvas, 无限画布, 图片迭代, 批注, MCP]
---

# 给 Codex 装上无限画布，AI 改图终于不用翻对话记录了

作者：林月半子

## 核心问题

AI 出图需要多轮迭代（改文案位置、换衣服、调眼镜……），但对话记录里的版本完全不可回溯，改到第五版就找不到哪个版本是哪条消息生成的了。

## Prompt Canvas 方案

以 Codex 插件形式运行，核心设计：

### 三层架构

**第一层：版本管理**
每张图片有版本号（v1、v2、v3），在画布上并排展示，一眼可见迭代过程。

**第二层：本地持久化**
SQLite 存所有数据（画布状态、图片版本、批注记录），关掉 Codex 再打开还在。

**第三层：批注工作流（核心）**
画布上方按钮：「批注」「备注」「复制批注指令」「提交给 Codex」。在图上画圈写文字，点「提交给 Codex」→ 批注变成结构化指令发给 Codex。

### 两种迭代方式

**方式一：截图丢给 Codex（最快）**
点截图按钮截当前画面丢给 Codex，自然语言告诉它哪里要改。

**方式二：批注后提交（最精确）**
在图上画箭头/画笔圈/文字，批注被结构化成 JSON，含 target、annotations（kind、region、text）、next_version、md 等字段。笔批注变成 annotations 数组里的记录，包含什么类型、什么区域、什么修改意见。

### 异步协作设计

用户批注 → 生成 修改说明书（.md + .json）→ 存到 `_pending/` 文件夹 → Codex 通过 MCP 工具 `prompt_canvas_get_pending_submits` 轮询 → 拿到后生新图 → 回填画布 → 标记已处理（移到 `_done/`）。

核心优势：**异步解耦**。你提交时 Codex 不一定在线，请求写成文件随时可取，相当于本地消息队列。.md 给 Codex 看，.json 给自动化流程用。

### 与 Cowart 的区别

- Cowart：截图让 AI 看（视觉模糊）
- Prompt Canvas：结构化指令让 AI 读（精确）
- Prompt Canvas 更偏生产管线，核心在版本管理和结构化批注指令

## 安装方式

1. `codex plugin marketplace add lqshow/prompt-canvas`
2. `npx skills add lqshow/linyuebanzi-skills -g`（生图依赖）
3. Codex 插件面板安装
4. 在 Codex 里说「打开一个新的画布」

## 开源地址

github.com/lqshow/prompt-canvas
