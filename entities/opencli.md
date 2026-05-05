---
title: OpenCLI
created: 2026-05-05
updated: 2026-05-05
type: entity
tags: [tool, open-source, cli, browser]
sources:
  - 知识库/技能文档/OpenCLI - 浏览器桥接多平台命令行工具-2026-03-27.md
confidence: high
---

# OpenCLI

## 概述

OpenCLI 是一个开源命令行工具（GitHub 7.3k ⭐），通过连接本地 Chrome 浏览器，复用已登录的会话状态，将任何网站变成终端可操作的命令。

- 项目地址：https://github.com/jackwener/opencli
- 龙虾技能包：https://github.com/joeseesun/opencli-skill
- 支持 44 个平台，244 个命令

## 工作原理

不走 API，不申请资质。直接连接本地已打开的 Chrome 浏览器，复用浏览器中已登录的会话状态。你能在浏览器里操作的平台，OpenCLI 就能操作。

## 安装条件

1. Agent 必须部署在本地电脑（非云端）
2. 安装 OpenCLI Skill
3. 安装 Chrome 扩展：OpenCLI Browser Bridge
4. 目标平台需先在 Chrome 中登录好账号
5. 默认浏览器必须是 Chrome

## 支持平台示例

| 类别 | 命令示例 |
|------|---------|
| 热门排行 | `opencli zhihu hot`, `opencli bilibili hot`, `opencli weibo hot` |
| 搜索 | `opencli bilibili search --keyword "..."` , `opencli xiaohongshu search --keyword "..."` |
| 读取浏览 | 各平台内容阅读 |
| 互动操作 | 点赞、评论、转发等 |

## 相关实体

- [[openclaw]] — 通过 Skill 集成 OpenCLI 能力
- [[agent-reach]] — 另一条互联网平台访问路径
- [[bb-browser]] — 浏览器增强工具
