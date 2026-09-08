---
title: "给每个AI发一台独立电脑？聊聊4000星开源项目OpenBot"
source: "知识姬 Mina（知识发电机）"
source_url: "https://mp.weixin.qq.com/s/gyxgfQ_SprHOQpvbHCrbuw"
date: "2026-09-03"
tags: [OpenBot, Agent安全, 沙箱, CEL策略, 审计, 人机接管, AG-UI, 开源]
---

# OpenBot：给每个 Agent 一台带安全网关的独立"虚拟电脑"

> **一句话**：CopilotKit 开源的 4000+ 星项目——给每个 Agent 分配独立容器（专属 Chromium + 文件空间 + 可选 gVisor 硬件隔离），并在工具调用链路上焊死一道安全网关（CEL 策略 + Fail-Closed + 前置审计日志）。MIT 完全开源可自托管。

## 解决的痛点

让 AI 跑终端、翻网页、改文件——怕幻觉搞崩系统。以前要么锁在纯对话框（只能吐代码自己复制跑），要么套个简陋 Docker（黑盒操作等报错看日志）。OpenBot 务实切入：**给每个 Agent 分配独立"虚拟电脑" + 焊死安全网关**。

## 五个工程细节

### 一、给 Agent 一台带桌面的"专属工作电脑"
Supervisor 模块给每个 Bot 单独起容器：
- **独立 Chromium 实例**：专属浏览器 Profile 和登录状态，互不污染
- **独立文件空间**：挂载 /workspace 卷，下载/报表全在隔离区
- **系统级沙箱**：底层支持时可配 gVisor（runsc）硬件级隔离
- 效果：Agent 之间、Agent 与宿主机之间网络和文件物理隔离——玩脱了也只影响那一个环境

### 二、不讲信任，只讲"先验策略"与审计日志（核心）
传统链路：LLM 输出调用指令 → 代码层直接执行 → 结果回传。OpenBot 改变链路：
1. **必须先过安全门禁**：碰触浏览器/文件/Shell/MCP 插件的请求先送 Server Gateway
2. **CEL 策略评估 + Fail-Closed**：CEL（Common Expression Language）规则校验动作——可细化到 URL 域名、Shell 命令白名单、页面特定元素；**默认拒绝**（没明确放行一律拦截，规则解析报错也拦截）
3. **前置写日志再做真实动作**：审核通过后**先向 PostgreSQL 写审计记录，才向沙箱发指令**——没有一次工具调用能绕过审计悄悄执行
- `/admin/audit` 面板：哪分哪秒点了哪个网页、执行了哪句 bash、哪项操作被策略阻断，一目了然

### 三、人类接管：Take the Wheel（遇到验证码别瞎猜，"换我来"）
- Agent 遇 Cloudflare 盾/图形验证码/2FA 时**主动发起求助**（不再反复重试烧额度）
- 前端 Channel 实时投射 Agent 屏幕，人点击**接管鼠标键盘**，处理完交还
- **接管期间 Agent 自动化被强制拒绝**——防止"人机抢鼠标"
- 接管与释放动作完整记入审计流水
- 评价：承认 AI 局限 + 人机协作切面顺滑，远比强称"完全全自动"理智

### 四、AG-UI 生成式界面（不吐 Markdown）
- 遵循开源协议 AG-UI，引入 Generative UI：Agent 输出图表/列表/审批流时**调用预注册 React 组件原地渲染可交互控件**（财报 → 可筛选缩放的交互图表）
- 基于 AG-UI 解耦：LangGraph/CrewAI/Mastra/Pydantic AI 底座，只要暴露标准 AG-UI 端点就能接入这套沙箱+审计+界面环境

### 五、`prompt.txt`：用 AI 部署 AI
- 仓库根目录放校对过的指令文件——丢给 Cursor/Claude Code 等，AI 按提示一步步带跑：查 Bun 依赖、拉 Docker、配三项必填环境变量，全自动跑通部署

## 结语判断

开源社区对 Agent 的探索方向明显变化：**从"套壳 Prompt"→"Multi-Agent 流程编排"→ 扎扎实实补齐操作系统层面的基础设施**。

> 没有权限隔离的 Agent 只是个玩具。能跑进生产环境的 Agent，必须拥有清晰的资源边界、可追溯的审计记录，以及随时能被人拉住的缰绳。

## 战略分析

**1. 与最近归档的"生产级 Agent 安全"谱系闭环：**

| 维度 | OpenBot | AgentDesk | Prime Agent | BrowserOS |
|---|---|---|---|---|
| 隔离 | 每 Agent 容器 + gVisor | 工作区/路径白名单 | ⚠️ 无沙箱（官方警告） | 本地双浏览器 |
| 门禁 | CEL 策略 + Fail-Closed | check_compatibility 写入门禁 | 无（靠人工） | - |
| 审计 | PostgreSQL 前置写日志 | Git 事件/outbox | 会话日志 | 操作录像 |
| 人接管 | Take the Wheel 抢盘 | 门禁处停下等人 | 人工校验门禁 | 录像复盘后叫停 |
| 界面 | AG-UI 生成式 | HTML 视图 | CLI | - |

**2. 关键洞察：**
- **"前置写日志再执行"与 AgentDesk"先提交意图再传输"（outbox 模式）完全同构**——跨项目验证的共识：**任何 Agent 动作先留痕后执行，是生产级信任的基础**
- **Fail-Closed（默认拒绝）**是最安全的设计哲学：没有明确放行就是拦截——与 AgentDesk 的 deny-by-default、TokBrain"权限不明就不下载"一致
- **Take the Wheel 是"人留关键节点"的物理实现**：验证码/2FA 正是"该人确认的节点"——不硬撑全自动，人机交接面做顺滑
- **prompt.txt 模式**（仓库内置 AI 部署引导文件）值得所有复杂项目借鉴——用户装 skill 也偏好这种"一键自动化"

**3. 与 Hermes 的关系：** Hermes 的 delegate_task 子代理是同一进程内隔离（无独立容器），terminal 工具直接宿主机权限——OpenBot 的容器隔离 + 前置审计是比 Hermes 更强的生产级安全层。对用户的启发：如果要用 Hermes 跑"脏活累活"（真实终端操作），参考 OpenBot 的思路——独立工作目录、命令白名单、操作留痕。

## 关联阅读

- [[2026-09-02-BrowserOS-带登录态本地AI浏览器]] — 同作者；登录态浏览器 vs OpenBot 容器内 Chromium
- [[2026-08-18-AgentDesk-任务卡驱动多Agent开发工作流]] — 写入门禁/outbox 先提交后发送同构
- [[2026-08-27-Prime-Agent-自进化Harness框架]] — 无沙箱 vs OpenBot 沙箱对照
- [[2026-08-18-轻量化业务应用-四步渐进式验证框架]] — 人留关键节点 = Take the Wheel 哲学

## 归档日志

- 2026-09-04 归档（Firecrawl cache hit；封面已存 assets）
