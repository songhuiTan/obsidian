---
title: Claude Code 开源编译版来了！45+实验功能全开，隐私无遥测，开发者终于等到了
source: https://mp.weixin.qq.com/s/V1T3BJe5PIF5ZutRHyTJPw
archived: 2026-04-02
tags: [Claude-Code, 开源, AI编程, free-code]
---

# Claude Code 开源编译版来了！45+实验功能全开，隐私无遥测，开发者终于等到了

Claude Code 的"开源编译版"真的来了。

3月31日，Anthropic 通过 npm 发行版中的源码地图（source map）意外泄露了 Claude Code CLI 的完整源代码。不到24小时，社区就基于这份快照打造出了 free-code——一个去除所有遥测、解除安全限制、解锁全部实验功能的完全体版本。

一条命令安装，45+ 功能全开。

## 🔥 free-code 是什么？

一句话：Claude Code 的完全解锁版。

它是 Anthropic 官方 Claude Code CLI 的开源编译分支，由社区开发者 paoloanzn 打造，在原版基础上做了三类核心改动：

**无遥测 · 无 guardrails · 45+ 实验功能全开**

安装只需要一行：

```bash
curl -fsSL https://raw.githubusercontent.com/paoloanzn/free-code/main/install.sh | bash
```

然后设置 API Key 即可使用：

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
free-code
```

## 🚫 第一刀：剔除所有遥测

官方 Claude Code 实际上会向外发送多种数据：

| 遥测类型 | 说明 |
|---------|------|
| OpenTelemetry / gRPC | 调用链路追踪 |
| GrowthBook Analytics | 功能开关分析 |
| Sentry Error Reporting | 崩溃报告 |
| 自定义事件日志 | 会话指纹等 |

free-code 把这些全部stub掉，所有外向端点 dead-code-eliminated 或替换为空实现。

GrowthBook 的本地功能开关特性依然工作（运行时需要），但不会回传任何数据。

结果：零回调，纯本地运行。

## 🔓 第二刀：移除安全提示 guardrails

这是 free-code 最具争议也最受关注的改动。

Anthropic 在 Claude Code CLI 中额外注入了一层 prompt 级别的安全限制，独立于模型本身的安全训练，包括：

- 特定类别 prompt 的硬编码拒绝模式
- 注入的"网络风险"指令块
- 从 Anthropic 服务器推送的托管设置安全覆盖层

free-code 移除了这些额外包装。模型本身的安全训练依然有效，只是把 CLI 加在模型外面的人为限制解开了。

> ⚠️ 这意味着模型的原生安全底线仍在，但 CLI 层面的额外限制不再生效。部分在官方版本中被"强化拒绝"的功能，现在可以正常触发。

## 🧪 第三刀：45+ 实验功能全部解锁

这是 Claude Code 被锁住的最大"宝藏"。

官方 npm 发布版中，大量功能被 feature flag 关闭，free-code 全部解锁了能正常编译的 45+ 开关。

### 🔑 核心亮点功能

**ULTRAPLAN — 远程多智能体策划**

在 Claude Code 网页版上调用 Opus 级别的模型进行远程多智能体协同规划。一个任务，多个 AI 代理同步策划，能力远超单代理。

**ULTRATHINK — 深度思考模式**

输入 ultrathink 关键词，模型自动提升推理投入，针对复杂问题展开更深度、更全面的思考。相当于 Claude 的"极限思考模式"。

**VOICE_MODE — 语音输入**

按住说话，语音直接转为指令。支持听写和快捷命令两种模式，编程时双手不离键盘。

**BRIDGE_MODE — IDE 远程控制桥**

打通 VS Code 和 JetBrains 系列 IDE，实现编程工具与 Claude Code 的深度联动控制。

**AGENT_TRIGGERS — 后台自动化触发器**

本地 cron 定时任务和触发器工具，Claude Code 可以在后台持续运行、响应定时事件。

**VERIFICATION_AGENT — 任务验证代理**

Claude 自动验证任务完成情况，给出结果检查报告，不用人工确认是否做对了。

**EXTRACT_MEMORIES — 自动化记忆提取**

每次对话结束后自动提炼关键信息存入记忆，跨会话积累上下文。

**TOKEN_BUDGET — Token 预算追踪**

实时监控 Token 消耗，设置预算上限和预警提示，避免意外爆费。

### 完整功能清单（部分）

| 功能 | 作用 |
|------|------|
| BUILTIN_EXPLORE_PLAN_AGENTS | 内置探索/计划代理预设 |
| BASH_CLASSIFIER | Bash 命令分类器辅助权限决策 |
| HISTORY_PICKER | 交互式命令历史搜索器 |
| QUICK_SEARCH | 提示词快速搜索 |
| COMPACTION_REMINDERS | 上下文压缩时的智能提醒 |
| CACHED_MICROCOMPACT | 查询流程中的缓存微压缩状态 |

完整 88 个开关的状态清单见 FEATURES.md。

## ⚙️ 技术架构一览

free-code 基于 Claude Code 官方源码构建，技术栈如下：

| 层级 | 技术选型 |
|------|---------|
| 运行时 | Bun |
| 语言 | TypeScript |
| 终端 UI | React + Ink |
| CLI 解析 | Commander.js |
| Schema 校验 | Zod v4 |
| 代码搜索 | ripgrep（内置） |
| 协议支持 | MCP、LSP |
| API | Anthropic Messages API |

## 📦 安装与使用

### 方式一：一键安装（推荐）

```bash
curl -fsSL https://raw.githubusercontent.com/paoloanzn/free-code/main/install.sh | bash
```

脚本自动检测系统环境，缺 Bun 就先装 Bun，克隆代码，全量编译，把 free-code 加到 PATH。

### 方式二：手动构建

```bash
git clone https://github.com/paoloanzn/claude-code.git
cd claude-code
bun install

# 全功能解锁版
bun run build:dev:full
```

### 使用

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
free-code

# 单次模式
free-code -p "列出当前目录的文件"

# 指定模型
free-code --model claude-sonnet-4-6-20250514

# 交互式 REPL（默认）
free-code
```

要求：
- macOS 或 Linux（Windows 可用 WSL）
- Bun >= 1.3.11
- Anthropic API Key

## 🗄️ IPFS 永久镜像

项目还有一个有意思的细节——完整代码库已在 IPFS 上永久钉住（通过 Filecoin），带 CID：

```
bafybeiegvef3dt24n2znnnmzcud2vxat7y7rl5ikz7y7yoglxappim54bm
```

即便 GitHub 仓库被下架，代码也会永久存在于分布式网络上。

## ⚠️ 需要注意

1. **API Key 自备**：免费安装，但需要自行申请 Anthropic API Key，有用量费用
2. **模型安全底线仍在**：移除的是 CLI 层面的额外限制，不是模型安全训练
3. **Windows 需 WSL**：原生支持 macOS / Linux，Windows 用户要走 WSL
4. **Linux 光标支持有限**：部分平台细节差异见官方文档

## 🔑 一句话总结

free-code 让 Claude Code 从"官方给你什么就用什么"变成了"全部功能、所有权限、零监控"。

对于重视隐私、不想被遥测、想要解锁全部实验特性的开发者来说，这条路终于通了。

**项目地址**：https://github.com/paoloanzn/free-code
