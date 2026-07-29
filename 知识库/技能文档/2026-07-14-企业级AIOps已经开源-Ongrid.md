---
title: "企业级 AIOps，已经开源！"
source: "PolluxAI"
source_url: "https://mp.weixin.qq.com/s/UYZlGMPquM7tKZUF_hcV8w"
date: "2026-07-14"
tags: [AIOps, 开源, Ongrid, 运维, Edge Agent, 可观测]
---

> Ongrid 是一个已开源的**企业级 AIOps 系统**，将监控发现异常到进入服务器调查的路径串成一条完整的 Agent 驱动流水线。

GitHub: [github.com/ongridio/ongrid](https://github.com/ongridio/ongrid)
文档: [ongrid.cloud/docs](https://ongrid.cloud/docs)

## 企业级 AIOps 不是聊天框

生产环境的核心挑战：
- Agent 看到的是否是当前用户有权访问的数据？
- 指标、日志、链路、拓扑、历史事故和代码如何进入同一份上下文？
- 工具运行在云端还是设备端，失败后如何继续？
- 读操作是否自动推进，写操作是否必须确认？
- 调查过程、命令原文和执行结果能否追溯？

Ongrid 把系统拆成**数据、Agent、知识、工具、执行和治理**几个相互配合的平面。

## 架构概览

```
用户入口：Web / 飞书 / 钉钉 / 企微 / Slack / Telegram
              │
         Control Plane
              │
    ┌─────────┼─────────┐
    │         │         │
Coordinator  Incident  Reviewer
(规划方向)  Investigator  (复核证据)
    │      (收集证据)      │
    │         │            │
    └─────────┼─────────┘
              │
    Specialist (SRE/网络/数据库/磁盘)
              │
    ┌─────────┼─────────┐
    │                    │
 Cloud Tools         Edge Agent
(Prom/Loki/Tempo)   (服务器现场)
```

- **Coordinator** — 理解目标、规划调查方向
- **Incident Investigator** — 围绕故障收集证据
- **Specialist** — 按问题类型（SRE/网络/数据库/磁盘）分发专业分析
- **Reviewer** — 复核证据、结论与高风险动作

## Edge Agent：从看见异常到进入服务器

Ongrid 的核心差异化能力：在受管主机上安装轻量 Edge Agent，主动向控制面建立长连接，主机**不需要额外开放 22/80/443 等入站端口**。

Agent 驱动的调查流程（以磁盘告警为例）：
1. 查询历史趋势和设备拓扑
2. 调用 `get_host_load` 获取 CPU/内存/负载快照
3. 调用 `get_host_processes` 检查资源占用最高进程
4. 调用 `host_du_summary` 逐层统计空间占用
5. 调用 `host_find_large_files` 定位异常大文件
6. 调用 `host_read_journal` 读取系统日志
7. 结构化工具不够时，通过受策略约束的 `host_bash` 完成灵活只读调查

**安全边界**：
- 结构化只读检查作为默认路径
- `host_bash` 默认受只读策略约束
- 写入操作需管理员显式开启总闸
- 变更命令生成确认卡（执行对象 + 命令原文），用户批准后执行
- 执行结果回到同一轮推理中继续验证

> 调查可以自动推进，变更必须受控执行；Agent 可以直接到现场取证，但不能绕过权限、审批和审计。

## Skills & MCP 工具目录

能力收敛为统一的 Skills 与 MCP 工具目录。每个工具带名称、参数、用途、运行位置和风险类型。Agent 只会看到当前身份、资源和策略允许的能力。

- 云端工具：Prometheus、Loki、Tempo、知识库、平台 API
- 设备端工具：主机现场检查（文件、进程、日志、网络命名空间）
- 外部 MCP：Grafana、K8s、GitHub、GitLab、PagerDuty

## Workflow 与知识沉淀

一次成功的人工调查可沉淀为 Workflow——告警触发、证据收集、Agent 分析、条件判断、风险闸门、报告生成和 IM 通知编排成可复用流程。

知识库通过 RAG 连接 Runbook、历史事故、架构文档和代码仓库——把"现在发生了什么"与"过去如何设计、类似问题如何处理"连接起来。

产物中心管理 RCA、容量分析、巡检、发布后健康检查等，默认私有，支持审阅、交接、分享。

## 为什么开源

AIOps 会接触企业最敏感的基础设施数据和操作入口。开源让企业能检查数据流向、Agent 编排、工具实现、主机通道、权限判断和执行逻辑，并能根据自己的组织边界裁剪能力。

---

## 战略分析

### 与现有体系的关系

**Ongrid 是目前见过的最完整的企业级 AIOps 开源方案。** 它的架构理念与我们的 Hermes Agent 体系有诸多共鸣：

| 维度 | Ongrid | 当前 Hermes 体系 |
|------|--------|-----------------|
| Agent 编排 | Coordinator + Investigator + Specialist 多角色 | 单 Agent 为主，delegate_task 可多 Agent |
| 工具目录 | Skills + MCP，云端/设备端区分 | Skills 体系 + 少量 MCP |
| 安全管理 | 只读默认、写入总闸、确认卡 | WeChat HITL 基础，未系统化 |
| Edge Agent | 主动长连接，无入站端口 | 不适用（本地运行） |
| 知识库 | RAG（Runbook/事故/文档/代码） | Obsidian 知识库 + 归档体系 |
| Workflow | 可复用自动化流程编排 | cronjob 定时调度 |
| 产物管理 | 报告/页面可管理 | 归档到 Obsidian |

### 值得借鉴的关键设计

1. **Edge Agent 的零信任连接模型** — 受管主机主动建连、无需开放入站端口，这个模式适用于任何需要在远端服务器执行 Agent 调查的场景。对我们来说，如果未来要管理远程服务器群，这是成熟可用的方案。

2. **"调查自动推进，变更受控执行"** 的安全边界定义非常清晰。Ongrid 对只读工具和写入工具有天然的分离策略，这是我们在 Hermes 工具分类中还未显式做的。

3. **一次调查 → Workflow 沉淀** — 从 prompt 驱动的临时调查到可复用的 Workflow，这个路径和我们的 skill 沉淀理念一致，但 Ongrid 有可视化的 Workflow 编排界面。

4. **告警/Webhook/定时任务触发的 Agent 调查** — 这和我们的 cronjob 机制类似，但 Ongrid 的触发链路更长：告警 → Coordinator 自动启动调查 → Investigator 收集证据 → Specialist 分析。

### 关联归档

- [[2026-07-03-开源AI运维项目被人卖200块入群费]] — 另一开源 AIOps 项目
- [[2026-05-27-如何搭建一套会思考的企业AIOps智能体平台]] — AIOps 平台方法论
- [[2026-05-26-运维人反内卷-ITOps-Agent-Platform-开源AI运维平台]] — 另一开源运维 Agent

## 归档日志

- 2026-07-16 归档（PolluxAI，Ongrid 开源公告，07-14 发布，07-16 修改）
