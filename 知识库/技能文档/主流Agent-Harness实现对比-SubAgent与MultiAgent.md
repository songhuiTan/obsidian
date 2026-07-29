---
source: 微信公众号
title: "主流Agent Harness实现对比——SubAgent与MultiAgent"
author: 孔某人（孔某人的低维认知）
date: 2026-06-10
url: https://mp.weixin.qq.com/s/FdaYXvEDr8YfALGDErdUfA
tags:
  - Agent Harness
  - SubAgent
  - MultiAgent
  - Claude Code
  - Codex
  - OpenAI Agents SDK
  - OpenCode
  - Kimi Code
  - OpenClaw
  - Hermes
  - Agent As Tool
---

# 主流Agent Harness实现对比——SubAgent与MultiAgent

> 孔某人「主流Agent Harness实现对比」系列第三篇。Claude Code有四种SubAgent模式（非Fork/Fork/Coordinator/Teammate），是目前最丰富的实现。Agent As Tool 才是真正普及的范式。
>
> 系列前篇：A1 Memory篇 / A2 Context压缩篇

## 一、SubAgent 的三个核心目的

1. **应对 Long Context 任务** — 拆解复杂任务到独立 Context Session，每个任务需要的最大 Context Window 变小，还能抑制模型在超长上下文下的偷懒/作弊
2. **获取旁观者视角** — 让另一个独立 Context Session 来检查/Review，旁观者视角有优势效果
3. **并行加速** — 可拆分并行子任务的任务可以加速

## 二、Claude Code（最丰富，4种模式）

### 2.1 非 Fork 模式（默认）
- 创建 SubAgent 时不继承主 Session 的 context，从零开始
- 5 种 Agent 类型：general-purpose / Explore / Plan / claude-code-guide / statusline-setup
- 支持前台/后台执行，后台完成自动通知，无需轮询
- 不允许 SubAgent 再创建 SubAgent
- **Agent As Tool**：SubAgent 的 prompt 必须自包含，像给新同事交代情况
- 返回结果对用户不可见，主 Agent 自行总结

### 2.2 Fork 模式（需环境变量开启）
- Fork 继承主 Session 的完整对话上下文
- **"不要偷看"** — 不要 Read fork 的中间输出，这抵消 fork 的意义
- **"不要抢跑"** — 不要捏造或预测 fork 结果
- 共享主 Agent 的 prompt cache，很便宜
- Fork Agent 启动时收到严格的 fork-boilerplate prompt（10 条不可商量规则）
- 输出格式：Scope: / Result: / Key files: / Files changed: / Issues:

### 2.3 Coordinator 模式（需环境变量开启）
- 主 Agent **特化为纯指挥者**，不能 Edit、不能 Bash，只能派 SubAgent、停 SubAgent、发消息
- Worker 结果以 `<task-notification>` XML 形式作为用户角色消息到达
- 主 Agent 负责 Synthesis（综合发现）— **永远不要写"基于你的发现"，那是懒惰委派**
- 并发规则：只读任务可自由并行，写密集任务同一组文件每次只跑一个
- 验证 = **证明代码能工作**，不是确认它存在
- 犯错时用 `TASK_STOP` 停掉 worker 再 `SendMessage` 续跑纠正

### 2.4 Teammate / Agent Swarms 模式（需实验性 flag）
- 更接近对等 Agent，每个 Agent 不是主 Agent 的 Tool
- 可异步独立长时间执行，相互之间使用信箱进行消息通讯

## 三、Codex

- **Collab 模式**（默认） — 类似 Claude Code 非 Fork，但 SubAgent 始终异步
  - 五个工具：spawn_agent + wait_agent + send_input + close_agent + resume_agent
  - 递归创建 Agent 默认 1 层
- **MultiAgentV2** — 类似 Claude Teammate，信箱通讯
- **Agent Jobs** — 批量模式，CSV 批量创建一组 Agent

## 四、OpenAI Agents SDK

- **Agent As Tool** — 类似 Claude Code 非 Fork，支持多层递归创建，权限请求层层穿透
- **Handoff** — 不是 SubAgent，而是直接替换当前 Agent 的 role/system prompt
  - 在原有 context 上继续执行，伪装成 Tool 调用触发
  - **无法复用 KV Cache**

## 五、OpenCode

- 仅支持 Agent As Tool（非 Fork 模式）
- 支持 task_id 恢复同一个 SubAgent 会话
- 并发启动多个 agent 以最大化性能

## 六、Kimi Code

- **Agent As Tool** — 前台和后台两种运行方式
  - 30 分钟超时，超时后恢复同一 agent
  - 不支持嵌套创建 SubAgent
- **AgentSwarm**（0.12.0 新增）— 批量模式
  - `prompt_template` + `items` 数组批量派发
  - 最多 128 个子 agent，自动排队

## 七、OpenClaw

- Agent As Tool，允许调用外部 CLI Agent 作为 SubAgent
- 支持 fork context / 持久性运行 session
- 支持多层嵌套创建 SubAgent
- 主 Agent System Prompt 可设定尽量委托而非自己动手

## 八、Hermes Agent

- **delegate_task** — Agent As Tool，支持嵌套
  - 推理密集 / 中间数据多 / 可并行 → 委派
  - 单步机械 / 单个工具调用 / 需用户交互 → 不委派
  - orchestrator 角色可进一步拆分任务给子 agent
- **Kanban** — 外部进程 Agent 调用，生命周期不绑定主 Agent
- **Mixture-of-Agents** — 同时调用多个不同模型生成再综合

## 九、整体趋势

- 目前主流更多使用 **Agent As Tool**（SubAgent 作为智能 Tool），而非古典对等 Multi Agent
- Claude Code 大量推行 SubAgent 大概从 Opus 4.6 开始
- Coordinator 模式（Agent 作为用户与 Agent team 之间的翻译层）可能是未来主流设计
