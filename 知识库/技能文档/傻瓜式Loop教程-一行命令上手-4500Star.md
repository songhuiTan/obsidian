---
source: 微信公众号
title: "傻瓜式Loop教程来了：一行命令直接上手，GitHub狂揽4.5k Star"
author: 量子位
date: 2026-07-03
url: https://mp.weixin.qq.com/s/EolKWeKXRi1EQS65uSYRYg
tags:
  - Loop Engineering
  - cobusgreyling
  - loop-init
  - 吴恩达
  - Claude Code
  - Codex
  - Agent 工程化
---

# 傻瓜式Loop教程来了：一行命令直接上手，GitHub狂揽4.5k Star

> 别再玩 Prompt 了，let's 拥抱 Loop。
>
> GitHub: github.com/cobusgreyling/loop-engineering（4.5k Star）

## Loop 五个基础元件

1. **自动化/调度** — 定时把该干的活儿找出来、分好类（每天、每5分钟等）
2. **工作树** — 建立隔离的并行执行环境，多个 agent 彼此互不干扰
3. **Skills** — 沉淀项目的固定知识
4. **插件和连接器** — 通过 MCP 接入真实工具（GitHub、Linear、Slack 等）
5. **子 Agent** — 将制作器和检验器分离
6. **记忆和状态** — 对话外长期存储

## 七套即用工作流

| 模式 | 用途 |
|------|------|
| daily-triage | 每日巡检 |
| pr-caretaking | PR 看管 |
| ci-cleanup | CI 清理 |
| dependency-scan | 依赖扫描 |
| issue-triage | Issue 处理 |
| merge-tech-debt-cleanup | 合并技术债后清理 |
| changelog-draft | 起草更新日志 |

## 一键启动

```bash
# 初始化（支持 claude/codex/grok/opencode）
npx @cobusgreyling/loop-init . --pattern daily-triage --tool claude

# Token 成本估算
npx @cobusgreyling/loop-cost --pattern daily-triage --level L1

# 就绪程度审计（0~100分 + 改进意见）
npx @cobusgreyling/loop-audit . --suggest

# 打 Loop Ready 徽章
npx @cobusgreyling/loop-audit . --badge
```

## Loop 成熟度三级

| 级别 | 行为 |
|------|------|
| L1 | 只发现问题、更新 STATE.md，不修改代码 |
| L2 | 允许在有验证器的情况下小范围自动修改，需人工审查 |
| L3 | 完整自动长时间运行 |

## 标准 8 步流程

定时触发 → 任务分诊 → 读取状态 → 创建独立工作区 → Agent 执行 → 验证器检查 → 连接 Git/工单系统 → 人工确认

## 吴恩达：三层 Loop

1. **Agent 编码 Loop**（最快，几分钟一轮）— Agent 写代码、测试、修改直到无 bug
2. **开发者反馈 Loop**（几十分钟/数小时一轮）— 人审查效果，提供反馈，Agent 进入下一轮
3. **外部反馈 Loop**（最慢，数小时/天/周）— 朋友、Alpha 测试、真实用户反馈 + A/B 测试

> 三层 Loop 由快到慢组成完整链路：**Agent 快速把东西做出来，开发者决定应该做成什么，用户证明它是否值得继续做。**
>
> 吴恩达认为，Loop 不会让人类退出软件开发——人类工程师的上下文优势（关于用户和产品的经验/品味），是 Loop 越循环越好的关键。
