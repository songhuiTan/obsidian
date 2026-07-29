# SxDevOps：AIOps 平台开源——可观测 + Agent 工作流

> 来源：凌云智算 · 2026-06-29
> 原文：https://mp.weixin.qq.com/s/xXg-trSgYQgyxae96J6dqg
> 分类：企业级 AI & AIOps

---

> 把可观测性、事件中心、任务中心、工单、容器和 RBAC 接成可确认、可审计的运维 Agent 工作流。模型负责理解，平台负责边界，人负责最终确认。

---

## 项目概况

- **GitHub**：https://github.com/aiyiyi121/sxdevops
- **在线体验**：https://www.sxdevops.top
- **产品介绍**：https://www.sxdevops.top/ai-agent-promo
- **开源协议**：Apache-2.0
- **技术栈**：Django + Django REST framework + Channels + Daphne（后端），Vue 3 + Vue Router + Pinia + Element Plus + ECharts + Vite（前端）
- **外部集成**：Kubernetes API、Docker、SSH、Prometheus/Grafana、SkyWalking、Tempo、Jaeger、Zipkin、Loki、ELK、SLS

---

## 核心定位

不是又一个 AI 聊天窗口，而是把 Agent **放进了真实运维平台里**：

- 能查告警、日志、链路、事件和任务
- 在执行动作前做权限校验、参数预检和人工确认
- 关键操作全程留痕可审计

设计哲学：**模型负责理解，平台负责边界，人负责最终确认。**

---

## 平台模块

| 模块 | 能力 |
|------|------|
| **可观测性** | 平台总览、系统态势、指标查询、日志检索、链路追踪、Grafana 看板、数据源管理 |
| **事件中心** | 失败事件、关键写操作、失败定位线索、复盘上下文沉淀 |
| **任务中心** | 主机任务、批量命令、脚本模板、任务草稿、执行历史、计划任务 |
| **工单系统** | 应用发布、审批流、SQL 审计、事务工单、变更留痕 |
| **容器管理** | K8s 集群、工作负载、Pod 终端、ConfigMap、Secret、Docker 环境 |
| **权限与审计** | 后端 API/前端路由/菜单/按钮/WebSocket 统一 RBAC |

---

## Agent 运行架构

不是"用户提问 → 大模型自由回答"，而是拆成多层：

### 1. MCP（工具层）
把平台能力变成 Agent 可调用的工具：查告警、查日志、查链路、查系统态势、查事件中心、生成任务草稿。

### 2. Skill / SOP（经验层）
沉淀团队排障经验。例如告警根因分析：先识别环境/服务/对象/时间窗口 → 交叉验证日志/链路/态势/近期变更 → 回答区分结论/证据/推断/建议动作。

### 3. Action Router（路由层）
判断用户意图，选择不同 Action。不同 Action 所需上下文、工具和输出结构不同。

### 4. Preflight（预检层）
执行前补齐上下文。缺关键字段时返回结构化确认，不让模型自己猜。

### 5. Agent Mode（执行模式）
- **Direct**：简单只读查询
- **ReAct**：边查边判断的告警根因分析
- **Plan + ReAct**：多步骤诊断、Runbook 生成、复杂协同任务

---

## 生产安全边界

| AI 可以做的 | AI 不能做的 |
|------------|------------|
| 告警根因初筛 | 直接删资源 |
| 日志/链路证据整理 | 直接重启生产服务 |
| 变更影响关联 | 直接执行高危命令 |
| PromQL/SQL/LogQL 候选查询生成 | 直接改数据库 |
| 巡检任务草稿生成 | 绕过 RBAC 和审批 |
| 故障复盘摘要 | — |

只读诊断直接返回事实；生成/写入/执行类动作必须：预检 → 确认 → 执行 → 留痕。

---

## 快速启动

```bash
docker compose up -d --build
```

访问：http://localhost:8000

---

## 关键认知

SxDevOps 的价值不在于让 Agent 更炫，而是把**权限、确认、审计、复盘**这些生产系统里真正麻烦的部分放进了平台设计里。AIOps 要落到生产工作流，需要的不是更强的模型，而是更严谨的平台边界 + 可沉淀的事件闭环。
