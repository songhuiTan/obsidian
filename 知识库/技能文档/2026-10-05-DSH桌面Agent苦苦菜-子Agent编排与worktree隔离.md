---
title: "把 DSH 改成自己的桌面 Agent，核心功能开源了"
source: "抖音 · 苦苦菜（Coding）"
source_url: "https://www.douyin.com/video/7693193486291569956"
date: "2026-10-05"
tags: [AI, Agent, DSH, DeepSeek-Harness, 桌面端, 开源, 多智能体编排]
---

# Corum-Harness：把 DSH 改成自己的桌面 Agent（苦苦菜开源版）

> 来源：抖音 · 苦苦菜（Coding）｜2026-10-05 10:40
> 原视频：[把 DSH 改成自己的桌面 Agent，核心功能开源了](https://www.douyin.com/video/7693193486291569956)（17 分 28 秒）
> GitHub：[kukucaiCndy/Corum-Harness](https://github.com/kukucaiCndy/Corum-Harness)（Apache-2.0，TypeScript，仓库 08-18 建、10-06 00:17 推送开源，归档日实测 11⭐/8 forks）
> 作者：苦苦菜（Coding），粉丝 8,366，获赞 23.3K｜GitHub：Kukucai（kukucai.cn）

![视频封面：把 DSH 改成自己的桌面 Agent](../assets/2026-10-05-DSH桌面Agent苦苦菜/cover.jpg)

## 概述

独立开发者苦苦菜花两个月把 **DSH（DeepSeek Harness）改造成自己的桌面端 Agent**，本期视频开源其核心能力并逐项讲解与原版的差异：**子 Agent 分工、指挥模式（贵模型规划+便宜模型执行）、编排工作流（Orchestrator）、git worktree 隔离**。当前只在 Mac 上测过，作者自述“边角不完善，欢迎试用、拍砖、共建”。

项目落地为 **Corum-Harness**：README 自述定位是"DSH 内核之上的发行版"——"DeepSeek Harness 是 Linux 内核（Cordis 插件框架 + agent loop + capability seam），本项目是发行版（产品化的 Agent 能力 + 插件 + 默认配置）。**不改内核，只做用户空间**"。构成 = dsh 运行时内核 + Electron 桌面承载层 + **38 个自研插件**（ui 12 / session 11 / agent 15 三组）+ 一套发行版 profile；必要的内核定制集中在 16 个 fork 包，差异登记在 docs/fork-delta.md 并用 scripts/verify-fork-drift.sh 守卫字节级一致性。

这是一条高信息量的"个人改造 DSH"实录：不是套壳 GUI，而是针对 DSH 多 Agent 并发时的真实痛点（文件冲突、上下文爆炸、昂贵输出浪费）做的机制级改动，每项都有演示佐证。

## 核心改动（相对原版 DSH）

### 1. 角色/模型双轨分离

| 角色 | 用途 | 配置 |
|------|------|------|
| 干活 Agent | 忠实执行者 | 每任务可单独配置模型 |
| 子 Agent（协调者） | 只读对话类模型 | 可设全局模型 |

主 Agent 与子 Agent 模型可设不同——原版 DSH 也有此机制但偏简陋，需手动配置环境变量。作者将其提升为 **Agent 预设（persona）**：岗位、人格、职责、域边界、模型路由（主/子/调查三档独立配置），注入 System Prompt。

### 2. 指挥模式（Composer）预设：只出方案不出代码

在 DSH 原有的交互模式基础上新增“指挥”预设：**只调研和规划、不输出代码**。理由：

- 输出 token 比输入贵一个量级（“白菜价模型的输出也比输入贵很多”）
- “代码输出是最不值钱的环节——没有 AI 时是这样，有 AI 以后还是这样”
- 让贵模型输出**系统性的、结构化的、思维性的东西**，代码交给便宜模型写，效果相当

README 里 Conductor Mode 的完整设计比视频更细：指挥者是"技术负责人，不是调度员"——判断问题是什么、该怎么解、结果是否真的达标；**恒只读**（不给写工具、只给只读 shell，用户权限档位只对 worker 生效）；**三阶段调研**（入门自己看（有界）→ 深入交给 research 子 Agent → 收尾只做点读验收）；**子 Agent 的报告是主张不是证据，验收必须亲自看 diff**。与 `standard`/`ptc`/`minimal`/`cordis` 并列为第五种 baseMode。

这本质是"大参数模型给方案、小参数模型执行"思路的产品化固化，与知识库已归档的 dsh-cortex（低成本多模型编排插件）同题。

### 3. Monaco 内嵌编辑器 + 可拆卸终端

- 内嵌 Monaco（VS Code 同款编辑器），可查看 Agent 改动 diff、跳转变量定义；部分 LSP 能力因未测试完整可能不可用
- 终端/编辑器面板可拖动、调整大小、拖成独立窗口——多屏场景把终端拉到另一块屏
- UI 引入“抽屉（drawer）”概念，面板可收纳到侧边

### 4. 编排工作流（核心差异点）

发布任务后由协调者（Planner）拆解并派发子任务：

- **强制前台调研**：派发的研究任务必须是前台阻塞任务，拿到结果才决定下一步——避免“后台主 Agent 幻想调研结果”的浪费
- **指挥模型全程只读**：协调者无文件修改权限，避免它绕过设计直接动手；它派不出任务时软件会自动再派一个有权限的子 Agent 补位
- **失败处理**：模型限流时弹卡片让用户选——临时换模型续命 / 永久改指模型 / 放弃本轮。“会话保留”，不因限流丢上下文
- 演示中真实翻车：两次 worktree 集成失败、模型超限中断，作者现场演示恢复流程——真实性加分

### 5. git worktree 隔离（解决并发文件冲突）

原痛点：DSH 主 Agent 派发多个子任务后，A/B/C 三个子 Agent 同时改同一个文件，互相看到对方的改动+冲突标记，反复横跳浪费时间，最后主 Agent 自己下场改，四个人都在改同一文件。

解法：每个并行任务跑在独立 git worktree 分支，完成后由集成者（Integrator）统一合并提交。作者类比团队开发：“大家拿同一份代码各自环境调试，最终提交后做统一的 Review——这是已被验证可行的方式，现在通过编排工作流固化进软件机制。”

### 6. 卡片式可视化

原版 DSH 输出是“一股脑全抛出”，信息密度低。改为卡片式：每张卡显示子任务状态、使用的模型、计划/结论，会话树可点开定位到具体节点。对“监控模型是否按我的想法执行”非常有用。

## 投入与状态

- 两个月、一人开发；打包链路当前仅 macOS（Apple Silicon），README 明示跨平台（Windows/Linux）尚在规划——视频里"Windows 找时间打包"与之吻合
- 工程要求：Node ^22.19.0 || >=24，pnpm 11.7.0，pnpm monorepo；`pnpm shell:dev` 开发态启动，`pnpm pack` 打 .app/.dmg
- 演示中承认存在缺陷（worktree 环境不干净导致两次集成失败；限流中断恢复手动）；README 自标 status: early development
- 归档日实测 11⭐/8 forks/0 open issues——刚开源一天的冷启动状态，但 README 完整度（fork 差异台账、漂移守卫脚本、CDP 实机验证技能、微信群+个人微信 hjkkclife 共建入口）远超同 star 级项目
- 仓库定位过程（方法论沉淀）：视频/评论区/作者签名均未给地址；先按"DSH desktop"等英文关键词扫 GitHub 未中——该仓库描述用"Deepseek-Harness 桌面版 Agent"全称且建库日早于常规扫描窗口两天；最终由小谭提供项目名 corum-harness 直接定位。教训：中文项目先用中文名/拼音名搜 owner，再翻 owner 仓库列表
- 评论区实录："dsh一升级全废"（DSH 升级即破坏依赖的生态位风险——README 的 fork-delta 守卫正是对冲这个）、"Windows 版有吗"（Mac-only 印证）

## 战略分析

**定位：DSH 生态“编排层”个人作品，与桌面套壳路线正交。** 2026-08 以来 DSH 桌面端生态爆发：dataelement/dsh-desktop（12K⭐）、anywhere-labs/dsh-desktop（30K⭐）、dsh-tauri（3K⭐）等都是“把 DSH Web UI 装进壳”的发行版路线；苦苦菜这个项目走的是另一条路——**不改壳，改机制**：指挥/执行分离、worktree 隔离、编排工作流都是对 DSH Agent 协作模型的重新设计。两条路线的对照价值在于：发行版解决“装起来麻烦”，编排层解决“多 Agent 用起来乱”。

**对本机/现有栈的直接启示（比项目本身更值钱）：**

1. **指挥模式是 token 经济学实践**：贵模型只读+只出方案，便宜模型写码。与知识库 09-02《DeepSeek-Harness缓存优化》的“输入便宜输出贵30倍”形成完整闭环——前者省输出费，这篇省的是“谁配输出”。Hermes 的多模型分层（subagent tier）已在做同类事，但“只读协调者”这个权限设计值得借鉴：协调者无写文件权限，从根上防越权。
2. **worktree 隔离是并行子 Agent 的基础设施**：Hermes 的 delegate_task 并行子代理如果将来跑代码任务，同样会遇到“两个子代理改同一文件”问题。git worktree + 集成者合并的协议可以直接搬。
3. **“强制前台调研”反幻觉设计**：调研任务必须阻塞等结果，不允许后台 Agent 编造调研结论再叠加浪费。对任何 orchestrator 都适用。

**风险与判断：** 个人作品、Mac-only、early development、演示中多次翻车、11⭐冷启动。但工程质量信号强：16 fork 包的差异台账 + 字节级漂移守卫 + CDP 实机三层验证，是单作者项目里少见的纪律性。机制思路 > 成品可用性——借鉴设计为主；macOS 本机满足运行条件（Node 22/pnpm 可装），想实测可直接 `pnpm shell:dev`。对 DSH 升级破坏依赖的风险，作者用 fork-delta 守卫对冲，但 38 插件 × 16 fork 的维护面仍是单点。

## 归档日志

- 2026-10-06 归档（Firecrawl 抓取页面元数据+章节要点；douyin_tiktok_scraper 签名端点 403，改走浏览器 performance 资源记录提取音频直链 → faster-whisper tiny 转录 499 段 17.5 分钟；封面已存；作者签名经页面内 fetch aweme detail API 取得；仓库由小谭提供项目名 corum-harness 定位，gh api 拉取元数据+README 补全，视频口径与 README 逐条对上）
