# Zread CLI - 本地代码库解读工具

> 来源：微信公众号「逛逛GitHub」| 发布时间：2026-04-03
> 原文：https://mp.weixin.qq.com/s/4UWurhgVGWvNrji-nT5vpw

## 简介

Zread CLI 是智谱 AI（zread.ai）推出的命令行工具，能自动分析本地项目代码，生成结构清晰的项目文档。适用于：
- 快速理解陌生项目的代码结构
- 给本地项目沉淀基础文档（替代手写 README）
- 给 AI Coding 工具（Cursor、Claude Code 等）提供项目上下文

## 安装

```bash
# 方式1: npm
npm install -g zread_cli

# 方式2: Homebrew
brew tap codegeex/homebrew-tap
brew install zread
```

首次运行会引导登录和默认配置。

## 使用

```bash
cd /path/to/your/project
zread              # 交互式解读
zread browse       # 查看生成的文档
```

交互式操作界面，方向键+回车完成操作。

## 产品定位对比

| 产品 | 场景 | 语言 |
|------|------|------|
| DeepWiki | 公开仓库解读 | 英文为主 |
| Zread 网页版 | 公开仓库解读 | 中文 |
| **Zread CLI** | **本地代码库解读** | 中文 |

## 适用场景

- 本地任意代码目录（公司内部项目、个人项目、clone 的开源代码）
- 可接入 Agent，让 AI 先用 Zread 生成项目文档再开发
