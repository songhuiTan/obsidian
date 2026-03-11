---
title: OpenClaw + Claude Code 全栈开发技能
tags: [OpenClaw, Claude-Code, AI开发, 全栈开发, 自动化]
created: 2026-03-11
---

# OpenClaw + Claude Code 全栈开发技能

## 📝 文章摘要

原文链接：https://mp.weixin.qq.com/s/Nlu_8FZZ2y-6Z7eZZbABtQ

**核心观点**：OpenClaw作为项目经理，Claude Code作为开发工程师，通过协作实现完全自动化的全栈开发流程。用户只需描述产品需求，OpenClaw自主完成PRD编写、技术方案设计、代码开发、测试验收全流程。

---

## 🎯 核心理念

### 角色分工

- **OpenClaw（项目经理）**：
  - 24小时在线，多平台支持
  - 持久记忆，任务管理
  - 拆解任务，协调进度

- **Claude Code（高级开发）**：
  - 编程能力碾压级
  - 理解整个代码库
  - 自主调试修复
  - 完整工具链

### 工作流程

```
用户描述需求
    ↓
OpenClaw编写PRD+技术方案
    ↓
用户确认
    ↓
OpenClaw指挥Claude Code开发
    ↓
自动化测试
    ↓
交付结果
```

---

## 💡 实战案例：TikTok爆款分析系统

### 产品功能

上传一个TikTok视频，自动：
1. 分析营销策略
2. 逐帧拆解镜头
3. 逆向生成Sora提示词
4. 语音转结构化脚本

### 开发效率

- 从需求到可用产品，全程自动化
- 用户只需参与两个节点：确认方案、看最终效果
- 中间开发过程在后台静默完成

---

## 🔧 技术核心

### 三大痛点及解决

#### 1. 进程管理空白
**问题**：Claude Code子进程可能因内存不足、网络断开、上下文过长等原因死掉，OpenClaw不知道

**解决**：`background:true` 后台运行，OpenClaw继续处理其他任务

#### 2. 交互阻塞卡死
**问题**：Claude Code经常停下来问问题，没人回答就永远等着

**解决**：`--permission-mode acceptEdits` 不用每改文件都停

#### 3. 结果送不回来
**问题**：Claude Code干完了，结果在终端里，没人知道

**解决**：OpenClaw监控进程，自动获取结果并通知

### 核心命令

```bash
bash pty:true workdir:~/projects/xxx background:true \
  command:"claude --session-id xxx --permission-mode acceptEdits '任务指令'"
```

**关键参数说明**：
- `pty:true`：伪终端模式，否则CLI会挂起或输出乱码
- `background:true`：后台运行
- `--session-id`：保持会话上下文，可resume恢复
- `--permission-mode acceptEdits`：自动接受编辑

---

## 📦 开箱即用配置

### 第一步：修改AGENTS.md

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

### 第二步：修改TOOLS.md

```markdown
## 开发工具

- Claude Code：已安装，用 OAuth 登录
- 项目目录：~/projects/
- Playwright：用于端到端测试
- FFmpeg：视频处理
```

### 第三步：修改USER.md

```markdown
- 偏好：我只说产品需求，技术方案你自己定。别问我技术细节。
```

**这条很重要！不写的话OpenClaw会不停来问你技术问题。**

---

## 🚀 核心Skill文件结构

### 文件位置
```
skills/fullstack-dev/SKILL.md
```

### 完整流程框架

```markdown
---
description: "全栈项目开发技能。当用户描述一个想做的产品/网站/工具时自动触发。你负责从需求理解到开发执行、测试验收的全流程。用户只需要描述想法，你自主完成一切。"
---

# Fullstack Development Skill

你是一个资深技术合伙人。用户是产品负责人，他只说想要什么，剩下的全部由你搞定。

你有一个高级开发工程师：Claude Code。你通过命令行指挥它写代码。

## 核心原则

## 完整流程

## Phase 1：需求理解 → PRD
## Phase 2：项目初始化
## Phase 3：逐功能开发
## Phase 4：自动化测试
## Phase 5：交付报告
```

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

## 🎓 经验总结

### 为什么用OpenClaw包装Claude Code

1. **OpenClaw是项目管理专家**：24小时在线、多平台、持久记忆
2. **Claude Code是代码专家**：编程能力碾压、理解代码库、自主调试
3. **角色清晰**：让专业的人做专业的事
4. **自动化闭环**：从需求到交付，无需人工干预

### 关键成功因素

- ✅ 使用PTY模式调用Claude Code
- ✅ 使用background模式后台运行
- ✅ 设置acceptEdits避免交互阻塞
- ✅ 明确用户偏好，减少技术问题询问
- ✅ 自动化测试和截图反馈

---

## 🔗 相关资源

- 原文：https://mp.weixin.qq.com/s/Nlu_8FZZ2y-6Z7eZZbABtQ
- 公众号：饼干哥哥AGI
- 核心文档回复关键词：Claude code

---

## 📌 归档标签

`#AI开发` `#OpenClaw` `#ClaudeCode` `#全栈开发` `#自动化` `#项目管理`
