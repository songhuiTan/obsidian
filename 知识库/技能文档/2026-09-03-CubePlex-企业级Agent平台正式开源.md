---
title: "CubePlex -- 企业级 Agent 平台正式开源"
source: "Chris（CubePlex 团队）"
source_url: "https://mp.weixin.qq.com/s/hj7jtM3su76GcX2yr8yuRA"
date: "2026-09-03"
github: "https://github.com/cubeplexai/cubeplex"
tags: [AI, Agent, 开源, 企业级Agent, Harness, MCP, Skills, Memory]
---

# CubePlex — 企业级 Agent 平台正式开源

> 来源：Chris（CubePlex 团队公众号）｜2026-09-03
> 原文：[CubePlex -- 企业级 Agent 平台正式开源](https://mp.weixin.qq.com/s/hj7jtM3su76GcX2yr8yuRA)
> GitHub：[cubeplexai/cubeplex](https://github.com/cubeplexai/cubeplex) · [cubeplexai/cubeloop](https://github.com/cubeplexai/cubeloop)

## 概述

CubePlex 正式开源。定位：**面向团队工作区的企业级智能体平台**，用 Workspace 集中管理成员、Agent 的 Skills、Memory 和 MCP 工具，通过 Cloud Harness 与隔离 Sandbox 运行任务。

写作背景是团队的真实痛点：2026 年初 OpenClaw 火了以后，公司里出现一批"养虾"的同事——个人 Agent 越来越好用（Hermes Agent、WorkBuddy 等持续演进），但进入团队后缺一套稳定的组织方式：谁来维护升级 Skills、MCP 连接谁配置、普通成员能用哪些能力，不能靠每人各自维护一份配置。这正是 CubePlex 要解决的问题。

**仓库热度（2026-09-17 GitHub API 核查）**：

| 仓库 | Stars | Forks | 创建 | 最近推送 | License |
|------|-------|-------|------|----------|---------|
| cubeplexai/cubeplex | 411 | 31 | 2026-03-04 | 2026-09-16 | Community License（Apache 2.0 + 附加条款） |
| cubeplexai/cubeloop | 59 | 11 | 2026-05-09 | 2026-09-16 | MIT |

注意：cubeplex 仓库创建于 2026-03（早于开源公告半年），是先内部打磨后开源，非蹭热点项目。

## 核心架构：Workspace + Cloud Harness + Sandbox

| 组件 | 职责 |
|------|------|
| **Workspace** | 组织单位。Admin 维护 Skills/Memory/MCP/成员权限；Member 直接使用已配置的 Agent，无需装依赖、不接触服务凭据 |
| **Cloud Harness** | 控制面持有 Agent loop 和会话上下文：模型请求、工具调度、审批。等待用户/定时/外部事件时，长期状态留在控制面 |
| **Sandbox** | 提供 Shell、文件、浏览器、本地进程，需要执行时才参与一次运行。故障不丢已完成步骤 |
| **CubeLoop** | 自研 Agent runtime，替换了最初的 LangGraph（详见下节） |

**Skills 版本化**是团队经验管理的关键设计：先在一个 Workspace 试用内部 Skill，跑通后推广到更多工作区；管理员升级版本后，后续对话使用新版本。业务专家写方法，平台管理员发布维护，其他成员拿来干活——经验有了明确归属。

**Memory 三级范围**：个人偏好留给个人，项目事实由 Workspace 成员共享，公司约定放组织范围。Agent 每次工作时读取与当前用户和 Workspace 对应的内容。

## 为什么替换 LangGraph：CubeLoop

文章对 LangGraph 的批评与本知识库已归档的多篇质疑文方向一致：

- 通用 Agent 不提前画固定流程图——模型根据上下文选工具、读结果、再决策，图运行时（节点、状态通道、序列化）是不直接服务于任务的结构负担
- CubeLoop 核心是**一段可直接读懂的异步 while loop**：工具是普通函数，模型输出和工具结果走统一事件流
- 类型明确的 **Middleware**：模型调用和工具执行前后插入权限、审批、观测
- **Checkpoint 从 O(n) 到 O(1)**：LangGraph Checkpoint 随对话增长写入完整消息状态；CubeLoop 改 append-only，每步只写新增消息
- **Prompt Cache 保护**：工具先给紧凑目录、按需加载 MCP 工具；工具数组和系统 Prompt 运行中保持稳定，不因动态加载反复破坏缓存

CubeLoop 已独立开源（MIT），既是 CubePlex 的 Harness Core，也可作为 Python Agent 框架单独使用。

## 桌面 Agent 的企业问题

文章对"桌面 Agent 进企业"的矛盾分析值得记录：

- 员工电脑里有环境变量、访问密钥、合同文件——Agent 能访问什么、执行过什么、哪些高风险动作需要确认，无法靠每人各自维护桌面配置统一
- Skill 放在某同事电脑上，别人不知道拿到的是不是最新版；MCP 凭据个人保存，人员变动要重新交接
- Agent 做过的工作留在本地会话，另一成员接手只能重新恢复现场

CubePlex 的答案：入口留在员工常用的 IM（飞书/Slack/钉钉/企业微信/Teams/Discord）和 Web，运行状态和执行环境放到云端可管理的位置。每条消息属于真实用户，只能进入所属 Workspace、使用该身份获准的模型和工具；成员可被邀请进同一条对话共享消息历史和 Sandbox。

## 企业落地要点

- **部署**：Docker Compose 单机 / Helm 上 Kubernetes，接入自己的模型和内部服务
- **产物管理**：报告、表格、代码、网页作为版本化产物留在会话，其他成员可打开继续修改
- **自动化**：计划任务和 Webhook 在无人守着聊天窗口时启动同一套工作流
- **执行策略**：可阻止命令，或要求成员确认后继续
- **凭据治理**：K8s 部署下长期凭据留在 Sandbox 之外，只在获准的网络请求到达出口时注入真实值
- **自托管**是产品前提：企业可读完整源码，检查 Agent 怎么调模型、怎么存状态

## License 注意事项（开源 ≠ OSI 开源）

CubePlex 用的是自定义 **Community License**，不是标准开源协议：

1. **品牌条款**：不得移除/替换/遮盖前端 CubePlex logo 和版权声明，主界面 logo 必须留在左上角品牌位且不小于默认尺寸（About/Settings 页展示不算合规）
2. **`ee/` 目录为企业版组件**：源码公开但生产使用需商业授权，移动/重命名/编译后仍受企业版协议约束
3. 其余部分基于 Apache 2.0，可自托管、商用、二次开发（含付费部署/定制/托管服务）

结论：**open-core 模式，源码可用（source-available），不是 OSI 定义的开源**。生产采用需评估 ee/ 组件依赖度。

CubeLoop 是纯 MIT，无限制。

## 关键洞察

- 个人 Agent → 团队 Agent 的核心矛盾是**组织方式**（Skills 归属、凭据治理、经验交接），不是模型能力——这与"Harness Engineering"系列的判断互相印证
- **Harness 与执行分离**：控制面持有 Harness（Agent loop + 状态），Sandbox 只做执行——作者在《Managed Agents Harness 架构与选择》两种方案中选了"控制面持 Harness"而非"Harness 进 Sandbox 整体释放"
- CubeLoop 是 "LangGraph 不适合通用 Agent" 叙事的又一实证：无固定流程图的任务里，while loop + 事件流 + Middleware 比图运行时更直接
- O(1) append-only checkpoint 和 Prompt Cache 保护（工具目录紧凑化 + 按需加载）是长会话/多工具场景下实打实的成本优化
- 名字来自数学"立方复形"：方体沿面连接织出复杂而有序的空间——连接（经验可复用）+ 秩序（统一规则）

## 战略分析

### 与现有体系的对照

CubePlex 与本知识库现有工具链（Hermes Agent 单机 + TencentDB Memory 单用户 + Obsidian 个人库）正面相关，对照如下：

| 维度 | CubePlex | 现有体系（Hermes + TencentDB Memory + Obsidian） |
|------|----------|--------------------------------------------------|
| 定位 | 团队 Workspace 企业平台 | 个人 Agent + 个人知识管理 |
| Memory | 三级范围（个人/Workspace/组织）共享 | TencentDB 四层（L0→L3）单用户 |
| Skills 管理 | Workspace 集中、版本化发布、Admin/Member 分权 | ~/.hermes/skills 本地、个人维护 |
| 执行 | Cloud Harness + 隔离 Sandbox，凭据留沙箱外、出口注入 | 本机执行，凭据在本地 .env |
| Runtime | CubeLoop（自研，替换 LangGraph） | Hermes 内置 loop |
| 入口 | Web + 飞书/Slack/钉钉/企微/Teams/Discord | Weixin 单端 |
| 部署 | Docker Compose / K8s 自托管 | 本机 macOS |

### 差距与借鉴点

CubePlex 补的正是现有体系缺的**组织层**：Skills 版本化发布流、Memory 共享范围、凭据治理、成员分权。目前的一人公司（OPC）多角色探索靠 Hermes 多角色架构支撑，CubePlex 提供了这些想法的工程化参照——特别是"业务专家写 Skill、管理员发布、成员使用"的分权模型，和多 Workspace 间先试用再推广的灰度思路。三级 Memory 范围（个人/团队/组织）对 TencentDB Memory 未来做多用户/组织化扩展是最直接的启发。

CubeLoop 单独价值高于平台本身：MIT 无限制，O(1) checkpoint、事件流、Middleware、Prompt Cache 保护这几个设计对任何自研 harness（包括理解 Hermes 内核）都有参照意义，与本库 20+ 篇 Harness Engineering 文档可互为印证。

### 成熟度判断

诚实结论：**早期项目，观察价值 > 直接采用价值**。411 stars / 31 forks / 1 watcher，开源公告刚发两周，社区生态未起。亮点是 repo 2026-03 就存在、持续推送到 9-16，工程节奏真实。企业 Agent 平台赛道已有多篇归档（Apboa、码神、ITOps 等），CubePlex 的差异化在"Workspace 组织 + Harness/Sandbox 分离"这个架构判断上，而非功能堆料。若要试，Docker Compose 单机起步成本低；生产采用前先确认 ee/ 组件依赖度。

## 相关文档

- [[2026-07-15-LangGraph-OpenAI-SDK-Skills-MCP都不是企业Agent最终答案]] — 同方向的 LangGraph 质疑
- [[2026-07-15-拆完WorkBuddy-我看到了生产级Agent的完整形态]] — 文中提及的个人 Agent 形态
- [[2026-06-16-Agent-Harness-Engineering生产级拆解]] — Harness 架构参照
- [[2026-05-19-开源-企业级可视化智能体平台-Apboa-Agent工程化解决方案]] — 同类企业 Agent 平台
- [[2026-07-18-Loopany-多用户定时Agent循环调度平台-BYOA-零执行架构]] — 多用户 Agent 平台对比
- [[2026-07-25-Harness引擎-自研编排工具-云淡风轻]] — 自研 runtime 替代大框架的同类案例

## 归档日志

- 2026-09-17 归档（Firecrawl 遇微信验证码 → 浏览器直接提取正文；GitHub API 核实双仓数据与 License 全文）
