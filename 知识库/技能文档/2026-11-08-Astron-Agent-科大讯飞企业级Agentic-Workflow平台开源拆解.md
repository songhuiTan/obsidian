---
title: "Astron Agent — 科大讯飞企业级 Agentic Workflow 平台开源拆解"
source: "GitHub / 科大讯飞 Astron 开源矩阵"
source_url: "https://github.com/iflytek/astron-agent"
date: "2026-11-08"
github: "https://github.com/iflytek/astron-agent"
tags: [AI, Agent, 开源, 企业级Agent, RPA, MCP, Workflow, 讯飞, SuperAgent]
---

# Astron Agent — 科大讯飞企业级 Agentic Workflow 平台开源拆解

> GitHub：[iflytek/astron-agent](https://github.com/iflytek/astron-agent)
> 官网/云服务：[astron.ai](http://astron.ai/) · [agent.xfyun.cn](https://agent.xfyun.cn)
> 文档：[DeepWiki](https://deepwiki.com/iflytek/astron-agent) · [中文 README](https://github.com/iflytek/astron-agent/blob/main/docs/zh/README.md)

## 概述

Astron Agent 是科大讯飞开源的企业级、商用友好（Apache 2.0）的 **Agentic Workflow 开发平台**，把 AI 工作流编排、模型管理、AI/MCP 工具集成、RPA 自动化、团队协作放在同一套体系里。定位一句话：**让智能体从"能聊"到"能在企业里真把事做完"**——不是又一个造 agent 的玩具，而是企业落地的生产级底座。

官方卖点四条：

1. **稳定可靠** — 与讯飞商业版 Astron Agent 平台同源核心技术，开源了完整的高可用版本
2. **跨系统集成** — 原生融合智能 RPA，把 agent 和企业的内部/外部系统接起来
3. **企业级开放生态** — 兼容各家行业模型与工具，支持自定义扩展
4. **商用友好** — Apache 2.0，无商业限制，可自由商用

**仓库热度（2026-11-08 GitHub API 实时核查）**：

| 仓库 | Stars | Forks | 创建 | 最近推送 | License | 语言 |
|------|-------|-------|------|----------|---------|------|
| iflytek/astron-agent | 8,921 | 910 | 2025-09-19 | 2026-11-06 | Apache-2.0 | Java |
| iflytek/skillhub | 5,375 | 817 | 2026-03-11 | 2026-11-06 | Apache-2.0 | Java |
| iflytek/astron-rpa | 5,256 | 484 | 2025-09-20 | 2026-11-06 | Apache-2.0 | Python |
| iflytek/astron-harness-tutorial | 360 | — | — | — | — | — |
| iflytek/astron-xmod-shim | 101 | — | — | — | — | Go |

- 83+ 位贡献者，头部贡献者 yjlu666 提交 881 次（核心团队驱动，非纯社区项目）
- 62 open issues / 77 watchers，仓库体积 115 MB
- 已收录进 **CNCF Landscape** 的 AI Agent Workflow Orchestration 分类
- 落地客户（README 展示）：中国电信、烽火通信、东华软件、福昕（厦门福昕中数）、云算数字、小趣科技、山东云谷、广物互联、北京榆构、襄阳东昇

> 注：公开传播口径称"9k+ stars"，实测 8,921，基本吻合；讯飞整个 Astron 开源矩阵合计约 20.7k stars。

## 它到底解决什么问题

平台围绕企业侧三个真实断点切入：

**1. 决策与动作断裂**
多数 agent 平台止步于"推理出结论"，但企业流程最后一步是去点 ERP、填 Excel、操作系统界面。Astron 的解法是**原生融合 RPA**——agent 里直接挂 RPA 工作流节点，补上"想明白"到"做出来"的最后一公里。这是它与 Dify/Coze 系最本质的差异点。

**2. 从 Demo 到生产的落差**
开源的是**与商业版一致的核心技术**，且号称完整开源高可用版本，所以带的是认证、多租户、配额、健康检查、网关鉴权、可观测性这一整套生产级组件，而不是能跑起来的 demo。

**3. 不锁模型、不锁工具**
模型层从 API 快速验证到私有化 MaaS 集群一键部署；工具层吃 MCP 生态 + 讯飞开放平台现成能力，免开发接入。

## 技术架构：模块化微服务 monorepo

控制台与核心能力解耦，前后端分离：

**Console**
- `console/frontend` — React 18 + TypeScript + Vite：智能体创建、聊天、工作流可视化编排、模型管理、插件市场
- `console/backend`（镜像名 `console-hub`）— Java Spring Boot：REST/SSE、鉴权、业务聚合

**Core 微服务（`core/` 目录）**

| 服务 | 技术栈 | 职责 |
|------|--------|------|
| `core/agent` | Python FastAPI | 智能体执行引擎，Chat / CoT / CoT-Process 推理模式 |
| `core/workflow` | Python FastAPI | 工作流编排、执行、调试、版本管理、事件流 |
| `core/knowledge` | Python FastAPI | 知识库、文档解析、向量化、RAG 检索 |
| `core/memory` | Python | 会话历史、短/长期记忆（v1.1 起接 Mem0） |
| `core/tenant` | **Go** | 多租户、空间隔离、组织管理、资源配额 |
| `core/plugin` | 混合 | `aitools` / `rpa` / `link` 三类插件服务 |
| `core-database` | — | 统一数据访问层 |

**通信拓扑**（仓库 `AGENTS.md` 明确约定）：
- 前端 → Console Backend：HTTP/REST + SSE
- Console Backend → Core 服务：HTTP/REST
- Core 服务 ↔ Core 服务：**Kafka 事件驱动**

**部署拓扑**（实测 `docker/astronAgent/docker-compose.yaml`，容器名 + 镜像）：

```
astro-agent 核心应用（ghcr.io/iflytek/astron-agent/*）
├── console-frontend       前端静态资源
├── console-hub            Java 后端
├── core-tenant            5052   Go 多租户
├── core-database          7990   数据层
├── core-agent             17870  Agent 运行时
├── core-workflow          7880   工作流引擎
├── core-knowledge         20010  知识库
├── core-link              18888  工具/MCP 接入
├── core-aitools           18668  AI 工具
└── core-rpa               17198  RPA 调度

中间件
├── postgres:14            sparkdb_manager
├── mysql:8.4.6            astron_console / agent / workflow / tenant / spark-link
├── redis:7                缓存 + 会话
├── minio                  对象存储（OSS_TYPE=s3）
└── nginx:1.25-alpine      统一入口

可选/认证（with-auth 变体）
└── casdoor:8000           统一认证（默认 admin/123，须改）
```

注意：Elasticsearch 7.16.2、Kibana、Kafka 3.7.0、Logstash 在 compose 里**默认注释掉**（`KAFKA_ENABLE=0`、`OTLP_ENABLE=0`），说明轻量部署有意裁剪了重组件。

**关键环境变量族**（`.env.example` 实测）：
- 模型接入：`LOCAL_MODEL_URL`（本地模型服务，指向 astron-xmod-shim）、`MAAS_*`（MaaS 平台部署/发布/克隆全流程）、`BOT_API_CBM_BASE_URL`（讯飞开放平台 ws）
- RPA：`RPA_URL` / `XIAOWU_RPA_TASK_CREATE_URL` / `XIAOWU_RPA_TASK_QUERY_URL`
- MCP：`LIST_MCP_PLUGIN_URL` / `RUN_MCP_PLUGIN_URL`
- 沙箱：`SKILL_SANDBOX_*`（产物桶、20MB 单文件、1GB 单工作流总量、扩展名白名单）、`CODE_EXEC_TYPE`（留空走内置 LangChain/Pyodide，按需切 E2B）
- 可观测：`OTLP_*`、`LANGFUSE_*`（默认关）
- 安全：`SKILL_SANDBOX_RUNTIME_CREDENTIAL_TOKEN`、`WORKFLOW_INTERNAL_API_KEY` 首次启动自动生成并持久化到私有 volume

## 版本演进（最能看出方向）

| 版本 | 时间 | 关键变化 |
|------|------|----------|
| v1.0.6 | 2026-04-30 | — |
| v1.0.7 | 2026-05-15 | Skill 脚本沙箱执行（首个第三方沙箱 E2B）、工作流定时调度、循环节点、产物文件收集 |
| v1.0.8 | 2026-05-29 | — |
| v1.0.9 | 2026-06-18 | 独立 Agent 运行时**重写为 Spring AI**（弃自研 OpenAI 兼容层）+ **MCP 支持** + Skill 模块（`read_skill_*` / `run_skill_*` 工具对） |
| v1.1.0 | 2026-07-06 | Agent 内可直接调用工作流、插件商店导入、**Mem0 记忆** |
| v1.1.1 | 2026-08-07 | — |
| v1.1.2 | 2026-09-07 | 最新 release |

主线很清楚：**从"工作流编排平台"转向"会自我加载技能、会记事的 SuperAgent 运行时"**。

## 生态矩阵（不是一个单仓项目）

讯飞打的是组合拳，三层对齐：

1. **astron-agent**（8.9k⭐）— 平台本体
2. **astron-rpa**（5.3k⭐）— Agent 可调的 RPA 套件，桌面端 Electron + Java + Python 引擎，20+ 组件包，Apache 2.0
3. **skillhub**（5.4k⭐）— 自托管技能注册中心（Java 21 + React 19），技能发布、命名空间、版本、RBAC、审计，可被 Astron Agent 装载运行，形成技能从开发到生产的全生命周期管理
4. **astron-xmod-shim**（101⭐，Go）— 声明式 AI 服务管控中间件，`DeploySpec`（用户意图）→ `GoalSet`（可验证收敛目标）→ Shimlet 插件对接 K8s/Docker，用于本地/私有化模型服务部署。带 Helm chart、Prometheus metrics、多架构镜像
5. **astron-harness-tutorial**（360⭐）— 教程型项目

## 部署方式

**方式一：Docker Compose（推荐）**

```bash
git clone https://github.com/iflytek/astron-agent.git
cd docker/astronAgent
cp .env.example .env
vim .env                                   # 参考 DEPLOYMENT_GUIDE_WITH_AUTH.md
# 先做 Casdoor 私有初始化/密码迁移（docs/CASDOOR_PASSWORD_STORAGE.md）
docker compose -f docker-compose-with-auth.yaml up -d
```

访问：前端 http://localhost/ · Casdoor 管理台 http://localhost:8000（**默认 admin/123 必须改**）

**方式二：Helm / K8s**
`helm/astron-agent` 目录已有完整 chart（`Chart.yaml` / `values.yaml` / `templates` / `tests` / `examples`），README 有中文部署指南，前置要求：可用 K8s 集群 + Ingress Controller（推荐 ingress-nginx）+ Helm 3。README 写"charts under development"，但仓库里已可用的状态。

## 值得注意的工程细节

- **Agent 运行时用 Spring AI 而非自研**：v1.0.9 主动放弃自研 OpenAI 兼容层，说明团队在"自研 runtime"和"用官方框架"之间选了后者——与同期 CubePlex 自研 CubeLoop 的选择相反，两种路线可对照看
- **沙箱双轨**：内置 LangChain/Pyodide 隔离沙箱为默认，E2B 按工作流单独开启；产物独立私有桶（与 console 桶分开），有单文件/单工作流容量上限和扩展名白名单
- **凭据治理**：内部 token 首次启动自动生成写入私有 volume，core-agent/core-workflow 只读挂载；显式覆盖时强校验长度并原子同步，避免"删掉环境变量导致旧 token 复活"
- **AGENTS.md 里的开发纪律**（对理解团队工程观有用）：跨服务改动必须先写明调用链和依赖方向，禁止无评估直接改共享层；涉及 Kafka/Redis/MinIO/鉴权先评估对其它服务的冲击；有官方 SDK 就必须用官方 SDK
- **自带 AGENTS.md / CLAUDE.md / .gemini**：仓库本身面向 AI 编码代理做了适配，属于"agent-native repo"的做法

## 与现有体系的对照

| 维度 | Astron Agent | CubePlex | 现有体系（Hermes + TencentDB Memory + Obsidian） |
|------|--------------|----------|--------------------------------------------------|
| 定位 | 企业级 Agentic Workflow 平台（重编排+重交付） | 团队 Workspace 企业 Agent 平台 | 个人 Agent + 个人知识管理 |
| 差异化 | 原生 RPA 融合、私有化 MaaS | Workspace 组织层、Cloud Harness/Sandbox 分离 | 本地执行、单端入口 |
| Runtime | Python Agent + **Spring AI** | 自研 CubeLoop（替换 LangGraph） | Hermes 内置 loop |
| 技能治理 | SkillHub 独立仓库、版本+RBAC+审计 | Workspace 集中 + 版本化发布 | ~/.hermes/skills 本地，个人维护 |
| 部署 | 15+ 容器 Compose / Helm | Compose / Helm | 本机 macOS |
| License | **Apache 2.0 纯开源** | Community License（带品牌条款） | — |
| 成熟度 | 8.9k⭐ / 83+ 贡献者 / 一年持续迭代 / 有企业客户 | 411⭐ 早期项目 | — |

## 成熟度与采用判断

**值得上手的场景**：企业内部系统级流程自动化（要碰票据系统、ERP、内部 OA 这类需要操作界面的系统）；需要私有化部署、数据与模型不出内网；需要能交付给客户的商业许可（Apache 2.0 是硬优势，CubePlex 那种带品牌条款的 Community License 在交付场景要额外评估）。

**需要犹豫的场景**：只做 LLM 应用层编排（Dify 更轻）；只想快速验证想法（直接上云 agent.xfyun.cn）；团队没有 Java/Go 运维能力（15+ 容器的栈运维成本不低）。

**一个必须清醒的点**：这是**讯飞商业平台的开源版**。Apache 2.0 是真的、技术是真的，但工具生态默认对接讯飞开放平台（`spark-api-open.xf-yun.com`），演进节奏与商业版绑定。83 位贡献者里头部 1 人提交 881 次，是核心团队驱动而非社区共建——这意味着路线图跟着讯飞到走，不会被社区投票改变。

**待验证项**：本库仅做静态调研（README + compose + env + 目录结构 + release 记录），未实际部署跑通。若要做深度评估，下一步应是本地起 Compose 栈，重点验三件事：① 非讯飞模型（如 DeepSeek/本地 Qwen）接入是否顺畅；② RPA 节点在没有讯飞账号时能否完全本地跑；③ 15 容器栈的实际资源占用。

## 相关文档

- [[2026-09-03-CubePlex-企业级Agent平台正式开源]] — 同类企业 Agent 平台，自研 runtime 路线对照
- [[2026-05-19-开源-企业级可视化智能体平台-Apboa-Agent工程化解决方案]] — 同类企业平台
- [[2026-07-27-StaffDeck-企业级AI数字员工平台-OpenBMB]] — 企业级数字员工平台
- [[2026-06-16-Agent-Harness-Engineering生产级拆解]] — Harness 架构参照
- [[2026-07-15-LangGraph-OpenAI-SDK-Skills-MCP都不是企业Agent最终答案]] — 框架选型争论（可对照 Astron 选 Spring AI 的决策）

## 归档日志

- 2026-11-08 归档（GitHub API 实时核查 5 个相关仓库元数据；拉取 README 全文中英文、`docker/astronAgent` 目录、`docker-compose.yaml` 服务清单、`.env.example` 全量变量、`helm/` chart 结构、`AGENTS.md` 开发约定、release/tag/contributor 数据。未实际部署验证）
