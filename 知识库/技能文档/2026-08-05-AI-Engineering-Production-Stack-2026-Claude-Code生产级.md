# AI Engineering Production Stack（2026）：用 Claude Code 搭建生产级 AI 系统

> 来源：AI研究生｜AI大模型观察站 2026-08-05
> 原文：[AI Engineering Production Stack（2026）：用 Claude Code 搭建生产级 AI 系统](https://mp.weixin.qq.com/s/qRL5bWAL3ATZcfFXw4Tsjw)
> GitHub：[moonpiee/The-2026-AI-Engineering-Production-Starter-Stack-with-Claude-Code](https://github.com/moonpiee/The-2026-AI-Engineering-Production-Starter-Stack-with-Claude-Code)
> 标签：AI, Claude Code, 生产级, Prompt, Harness, 多Agent, 成本控制

---

## 概述

AI Engineering 的变化速度极快：Prompt Engineering → RAG → agents → multi-agent → MCP → Claude Code → autonomous coding。工程师真正关心的问题：**如何让 agents 持续工作数小时而不丢失 context、不烧掉 token、不过度偏离任务？**

本文是"2026 如何成为 AI Engineer"路线图的实战版本，简化成**四个关键组件**，全部可直接在 Claude Code 中动手。

## 组件 1：Context & Memory Architecture

### CLAUDE.md（standing memory）

项目"始终成立"的信息。与 README 的区别：README 写给人类，CLAUDE.md 为 AI agent 优化（压缩规则、命令、代码风格偏好），每个 session 自动加载。

包含：build/test 命令、仓库结构、编码规范、命名标准、首选库、架构决策、团队实践、部署命令、重要约束。

**黄金规则：**
- 控制在大约 **200 行** 以内，超长把 workflow 相关移入 Skills（只在被调用时加载，省 token）
- **"如果它下个月仍然成立，就放进 CLAUDE.md"**

### CLAUDE.md vs Skills

| 载体 | 用途 |
|------|------|
| **CLAUDE.md** | Claude 应该**始终知道**的信息（项目规则） |
| **Skills** | 可复用的 **workflows**（Releases / DB migrations / Code reviews / Deployment / Incident response / Performance profiling） |

### progress.md（tracker）

记录超出单次 session 生命周期的工作状态（hand-off note）：
- What's done / What's blocked / What decision was made / Why

### /compact vs /clear

| 命令 | 场景 |
|------|------|
| **/compact** | 压缩当前 session 历史，继续工作（防止撞 context 上限，清掉过时噪音） |
| **/clear** | 完全无关任务之间使用（如刚做完 RAG 转去做博客）——**比 compact 便宜得多**，不把旧 context 硬带到新任务 |

## 组件 2：Opus 4.8 与 Dynamic Multi-Agent Workflows

大型陌生 codebase / 架构决策 / 跨文件重构：大多数团队首选 Claude Code 的 **parallel workflows**（claude-opus-4-8）。

### 两种编排模型

**Subagent**：独立 Claude 实例，拥有自己的 context window，通过 Task tool 启动。**不继承父级对话历史**（这正是重点）——只做一件事，返回干净结果。每个 subagent 得到狭窄、只读受限的 tool set，没有 Edit/Write，真正提交更改的只有 parent agent。

**Dynamic workflows**：orchestrator 在运行过程中**临时生成**工作，而不是预先 scripting 每个分支。由 orchestrator 决定：需要多少 agents、每个做什么、何时运行/停止。可生成一整棵 subagents 树（Claude 中最多嵌套五层）。

```
Root Orchestrator
├── Backend Agent ── Test Agent
├── Frontend Agent ── Test Agent
└── Payments Agent ── Test Agent
        └── Results Returned to Root
```

## 组件 3：Token Financial Management and Effort Control

### Effort 是独立于 model choice 的调节项

五个 effort level：Low / Medium / High / xHigh / Max。控制模型在行动前进行多少 reasoning。**同一任务，高 effort 可能是低设置的 7 倍 token 开销**——和 model-selection 一样重要。

### 工作分配模式（Role-Effort）

| 角色 | Effort |
|------|--------|
| **Root orchestrator** | `high`（决定下游正确性的决策） |
| **执行型 subagents** | `low` / `medium`（任务窄、定义明确，只需可靠低成本执行） |
| **一次性深度分析** | `ultrathink`（single-prompt keyword，不永久提高 session effort） |

**模型别名技巧：**
- `opusplan`：planning 阶段跑 Opus，执行阶段自动切换 Sonnet——plan 用顶级判断，执行不付 Opus 费用
- `ultracode`：xhigh effort + 多-agent workflows 持续权限，强大但仅限真正需要高自治度的任务

### 条件验证 > 全量验证

不要每一步都跑完整验证（重读 diff、全量测试）。只在：
- 步骤触及**共享状态**
- **跨了模块边界**
- subagent 自身 **confidence signal 较低**

时才触发昂贵检查。**只有确实需要时才跳过 full verification。**

## 组件 4：Ambient Execution and Safety Navigation

### /remote-control

把正在运行的**本地** Claude Code session 连接到 Claude mobile app 或浏览器——手机监控/引导长任务。session 一直在自己机器上运行，mobile/web 只是窗口。

### Automated PR handling

Claude Code 官方 GitHub Action，`@claude` mention 触发：
- 每次 PR push 自动 review，inline comments 按 confidence 评分（低信号噪音在到达人工前被过滤）
- issue 打标签时自动修复（建 branch → 实现 → 开 PR）
- **注意**：意味着 Claude 在 repo 上有 write permission——**有意识地限定 GitHub App 权限，不要默认宽泛授权**

### 避免 false-positive injection flags

Claude Code 会检测 prompt-injection patterns，但会误触。**解决：用自己的话说明想做什么、为什么**，不要组织得像 system-level overrides、adversarial framings 或嵌套 ignore previous instructions 模式。

## 入门参考结构

```
Step 1 Create CLAUDE.md
Step 2 Add progress.md
Step 3 Create agents
Step 4 Add Skills
Step 5 Add GitHub workflow
→ Production ready repository
```

## 常见错误清单

- 把 Claude Code 当 ChatGPT，而不是 engineering environment
- 把所有内容塞进 CLAUDE.md
- 每个任务都用 maximum effort
- 从不使用 /clear
- 跳过 progress.md
- 给每个 subagent 都开放 write access
- 在每个微小改动后都运行 expensive verification
- 对不受益于并行化的工作使用 multiple agents

## 战略分析

### 与现有体系的对照

| 维度 | 本文方案（Claude Code） | Hermes 对应 |
|------|------------------------|------------|
| Standing memory | CLAUDE.md（~200行） | AGENTS.md / 项目上下文 |
| Tracker | progress.md | 会话记录 / Todo |
| Context 管理 | /compact / /clear | 上下文压缩（部分） |
| 子代理 | Subagent（五层嵌套） | delegate_task 子代理 |
| 动态编排 | Dynamic workflows | 子代理树（部分支持） |
| 成本控制 | Effort 5 档 + opusplan/ultracode | 模型选择 + cron 预算 |
| 验证 | 条件验证 | 无显式 Gate 体系 |
| 远程控制 | /remote-control | 手机端 WeChat 控制（已有） |
| PR 自动化 | GitHub Action + confidence 过滤 | github-pr-workflow skill |

### 高价值启示

1. **"条件验证 > 全量验证"** 与近期归档的多个主题共振：SkillCorpus 的 Harness 差距（OpenClaw 写完不校验 vs Raven 完整闭环）、rgraph 的确定性验证、Harness 引擎的门控设计。**验证不是越多越好，是按需触发**——这是生产级 token 成本控制的关键。

2. **CLAUDE.md 200 行规则 + Skills 分离** 与 Hermes 的 AGENTS.md/SKILL.md 分层同构。记忆和 Skill 知识库中的 Trellis Spec 分层（按需加载）理念一致。

3. **"把 AI 当工程师，不是 autocomplete"** 是核心认知转变——未来 AI engineer 不是用最新模型/框架的人，而是知道怎么 design 出让 AI 高效工作的环境的人。

4. **/clear vs /compact 的成本意识**：不相关的任务之间用 /clear 比 compact 便宜——简单但常被忽视的 token 管理。

5. **GitHub Action confidence 过滤**：低信号噪音在到达人工前被过滤，人工 reviewer 专注于架构/设计/业务逻辑——与知识库已归档的 AI 代码评审（AWS AI-DLC、WorkBuddy reviewer）同向。

### 局限

- 文章基于 Claude Code 生态，Hermes 需映射到自己的工具（但底层原则通用）
- Effort 5 档 / opusplan / ultracode 是 Claude Code 特有命令
- GitHub Action 的 write permission 需要团队权限治理意识

## 归档日志

- 2026-08-05 归档
