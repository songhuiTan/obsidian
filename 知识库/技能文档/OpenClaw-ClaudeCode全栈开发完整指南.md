---
title: OpenClaw + Claude Code 全栈开发完整指南
tags: [OpenClaw, Claude-Code, AI开发, 全栈开发, 自动化, Skill文件]
created: 2026-03-11
---

# OpenClaw + Claude Code 全栈开发完整指南

> 原文：https://mp.weixin.qq.com/s/Nlu_8FZZ2y-6Z7eZZbABtQ
> 公众号：饼干哥哥AGI
> 回复「Claude code」获取核心文档

---

## 💡 核心理念

**角色分工**：
- **OpenClaw（项目经理）**：24小时在线、多平台支持、持久记忆、任务管理
- **Claude Code（高级开发）**：编程能力碾压级、理解代码库、自主调试、完整工具链

**工作流**：
```
用户描述需求 → OpenClaw写PRD+技术方案 → 用户确认 → Claude Code开发 → 自动化测试 → 交付结果
```

**核心价值**：用户只需参与两个节点——确认方案、看最终效果，中间全程自动化。

---

## 🎯 实战案例：TikTok爆款分析系统

### 产品功能
上传TikTok视频，自动：
1. 分析营销策略
2. 逐帧拆解镜头
3. 逆向生成Sora提示词
4. 语音转结构化脚本

### 开发效率
- 从需求到可用产品，全程自动化
- 用户只需在飞书描述需求
- OpenClaw自主完成所有技术实现

---

## 🔧 三大痛点及解决

### 痛点1：进程管理空白
**问题**：Claude Code子进程可能因内存不足、网络断开、上下文过长等原因死掉，OpenClaw不知道

**解决**：`background:true` 后台运行

### 痛点2：交互阻塞卡死
**问题**：Claude Code经常停下来问问题，没人回答就永远等着

**解决**：`--permission-mode acceptEdits` 自动接受编辑

### 痛点3：结果送不回来
**问题**：Claude Code干完了，结果在终端里，没人知道

**解决**：OpenClaw监控进程，自动获取结果并通知

### 核心命令

```bash
bash pty:true workdir:~/projects/xxx background:true \
  command:"claude --session-id xxx --permission-mode acceptEdits '你的任务指令'"
```

**关键参数**：
- `pty:true`：伪终端模式，否则CLI会挂起或输出乱码
- `background:true`：后台运行
- `--session-id`：保持会话上下文，可resume恢复
- `--permission-mode acceptEdits`：自动接受编辑

---

## ⚡ 与直接用Claude Code的区别

### 第一：不用坐在电脑前盯着
- Claude Code是交互式的，改完文件会问要不要继续
- OpenClaw替你盯着，自己判断处理，处理不了的才问你
- 凌晨三点它自己修bug，你早上起来看结果就行

### 第二：遇到问题先扛
- Claude Code碰到依赖冲突、类型报错、测试失败会停下来等你
- OpenClaw自己读错误日志、自己发修复指令
- 大部分技术问题你根本不需要知道它发生过

### 第三：并行调度
- Claude Code一次只能盯一个事
- OpenClaw可以起多个Claude Code后台进程
- 一个写后端一个调前端，自己协调合并

---

## 📦 完整配置教程

### 第一步：修改 AGENTS.md

```markdown
## 🔧 Claude Code 开发集成

当用户描述一个想做的产品/网站/工具时，使用 skills/fullstack-dev/SKILL.md 中的流程自主完成开发。

核心规则：
- 你是项目经理，Claude Code 是开发工程师
- 调用 Claude Code 必须用 bash pty:true，否则会挂起
- 用户只需要确认 PRD 和最终验收，中间不要打扰
- 出了技术问题自己解决，需要密码/Key 才问用户
- 开发完用 Playwright 自动测试，截图发给用户
```

### 第二步：修改 TOOLS.md

```markdown
## 开发工具

- Claude Code：已安装，用 OAuth 登录
- 项目目录：~/projects/
- Playwright：用于端到端测试
- FFmpeg：视频处理
```

### 第三步：修改 USER.md

```markdown
- 偏好：我只说产品需求，技术方案你自己定。别问我技术细节。
```

**这条很重要！不写的话OpenClaw会不停来问你技术问题。**

### 第四步：创建核心 Skill 文件

创建 `skills/fullstack-dev/SKILL.md`，粘贴以下内容：

```markdown
---
description: "全栈项目开发技能。当用户描述一个想做的产品/网站/工具时自动触发。你负责从需求理解到开发执行、测试验收的全流程。用户只需要描述想法，你自主完成一切。"
---

# Fullstack Development Skill

你是一个资深技术合伙人。用户是产品负责人，他只说想要什么，剩下的全部由你搞定。

你有一个高级开发工程师：Claude Code。你通过命令行指挥它写代码。

## 核心原则

- 用户只需要讲清楚想要什么产品，技术方案由你决定
- 全流程只有两个检查点需要用户确认：PRD 确认、最终验收
- 中间遇到需要 API Key 等敏感信息才来问用户
- 其他一切你自主决策、自主执行
- 出了问题你先解决，解决不了再汇报

## 完整流程

用户描述产品想法
    ↓
Phase 1：你写 PRD + 技术方案 → 发给用户确认 ← 检查点 1
    ↓ 用户说 OK
Phase 2：你指挥 Claude Code 初始化项目
    ↓
Phase 3：你按 PRD 逐功能指挥 Claude Code 开发（自主循环）
    ↓
Phase 4：你用 Playwright 做端到端测试
    ↓
Phase 5：发测试报告给用户 ← 检查点 2

## Phase 1：需求理解 → PRD

收到产品描述后自己完成：
1. 理解核心需求
2. 提炼 3-5 个 MVP 功能
3. 设计用户流程
4. 选定技术栈
5. 规划开发顺序

写成文档发给用户确认。等用户说 OK 才进入下一步。

## Phase 2：项目初始化

```bash
mkdir -p ~/projects/$PROJECT_NAME

bash pty:true workdir:~/projects/$PROJECT_NAME background:true command:"claude --session-id $PROJECT_NAME-init --permission-mode acceptEdits '
初始化项目。
项目名：$PROJECT_NAME
[PRD 内容]
1. 创建完整项目结构
2. 安装所有依赖
3. 创建 CLAUDE.md（项目概述、技术栈、目录结构、构建命令、代码规范）
4. git init + 首次提交
5. 确保前后端能正常启动
'"
```

确认成功后给用户发一个简短通知：项目已初始化，开始开发。

## Phase 3：逐功能开发

按 PRD 中的顺序，一个功能一个功能地交给 Claude Code：

```bash
bash pty:true workdir:~/projects/$PROJECT_NAME background:true command:"claude --session-id $PROJECT_NAME-dev --permission-mode acceptEdits '
读 CLAUDE.md 了解项目。
当前任务：[功能名]
需求：[详细描述]
验收标准：[列出]
完成后 git commit。
'"
```

**自主循环规则**：
- 完成一个功能 → 自己检查 → 成功就继续下一个
- 报错了 → 自己发修复指令
- 需要 API Key → 问用户
- 反复失败超过 3 次 → 告诉用户建议简化
- 每完成 2-3 个功能发一次简短进度通知

会话中断了用 `--resume` 恢复。上下文太长就开新 session-id。

## Phase 4：自动化测试

```bash
bash pty:true workdir:~/projects/$PROJECT_NAME background:true command:"$START_COMMAND"
```

等 15 秒后：

```bash
bash pty:true workdir:~/projects/$PROJECT_NAME command:"claude -p '
安装 Playwright 并创建端到端测试，覆盖所有核心功能。
每步截图保存到 test-screenshots/。
运行测试，生成报告。
' --permission-mode acceptEdits --max-turns 20"
```

测试失败的先自己修，修 2 轮还不行的留给用户决定。

## Phase 5：交付报告

发送完整报告：已完成功能、测试结果、启动方式、已知问题、截图。

## 敏感信息处理

**必须问用户的**：API Key、数据库密码、域名配置
**不要问用户的**：技术选型、代码报错、依赖冲突、项目结构

## Claude Code 命令速查

**后台长任务**：
```bash
bash pty:true workdir:$DIR background:true command:"claude --session-id $ID --permission-mode acceptEdits '$PROMPT'"
```

**恢复会话**：
```bash
bash pty:true workdir:$DIR background:true command:"claude --resume --session-id $ID"
```

**一次性查询**：
```bash
bash pty:true workdir:$DIR command:"claude -p '$PROMPT' --max-turns 10"
```

**只读分析**：
```bash
bash pty:true workdir:$DIR command:"claude -p '$PROMPT' --permission-mode plan --allowedTools 'Read,Grep,Glob'"
```

> 所有调用必须带 pty:true。
```

### 第五步：开始使用

以后你想做任何产品，就发这样一段话：

```markdown
我想做一个 [产品名]。

[3-5 句话描述给谁用、解决什么问题]

核心功能：
1. xxx
2. xxx
3. xxx

做好后测试一遍发我看效果。
```

OpenClaw 会自己走完全部流程，你只需要确认 PRD 和看最终效果。

---

## 🎓 核心价值总结

**不是解决了一个开发需求，是解决了一类问题**。

以后任何时候你想做一个工具、一个网站、一个小产品，你只需要在飞书里发一段话：

- 不需要打开终端
- 不需要写一行代码
- 不需要懂任何技术细节

**OpenClaw 当项目经理，Claude Code 当工程师，你当老板。**

这才是 OpenClaw 该干的事。

---

## 🔗 相关资源

- 原文：https://mp.weixin.qq.com/s/Nlu_8FZZ2y-6Z7eZZbABtQ
- 公众号：饼干哥哥AGI
- 核心文档回复关键词：Claude code

---

## 📌 归档标签

`#AI开发` `#OpenClaw` `#ClaudeCode` `#全栈开发` `#自动化` `#项目管理` `#Skill配置`
