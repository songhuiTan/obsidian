---
title: "[开源]基于 DeepSeek Harness/DSH 构建的可私有化部署企业 Agent 平台"
source: "IT运维技术圈（作者：静静和小沐沐）"
source_url: "https://mp.weixin.qq.com/s/4l5lCdoAvHX-xXPLKqHVkw"
date: "2026-09-15"
tags: [MOVO, DeepSeek-Harness, 企业Agent平台, 私有化部署, Skill治理, MCP, RAG, 多租户, 开源]
---

# MOVO：基于 DeepSeek Harness 的可私有化部署企业 Agent 平台

> **一句话**：MOVO（himovo/movo）不造新 Agent Runtime，直接把 DeepSeek Harness 当底层执行引擎，在上面补企业应用层——组织/用户/角色、模型管理、企业 Skill 共享、Tools/MCP 治理、知识 RAG、Token 统计、系统审计。Self-hosted 为主方向，Docker Compose 一键起 12 个服务。

## 概述

这半年 Agent 领域的明显变化：大家已经不太缺"能跑起来的 Agent 框架"。DeepSeek Harness 出来后，任务执行、工具调用、Skill、插件、Session、沙箱这些 Runtime 层能力已经完整。但 **Agent 能跑和 Agent 能进生产是两回事**——进真实企业环境马上遇到：用户怎么管、权限怎么控、知识怎么接、Skill 怎么共享、模型怎么管理、Token 怎么统计、操作怎么审计、内部系统怎么连接。

MOVO 的切入点：不做新 Runtime，把 DSH 作为底层执行引擎（"发动机"），自己专注 Enterprise Layer（"企业控制台和工作空间"）。

## 核心框架：Runtime 与 Enterprise Layer 分层

| 分层 | 职责 | 具体内容 |
|------|------|----------|
| DeepSeek Harness（DSH） | Agent 怎么运行 | 规划、执行、工具调用、Skill、插件、Session、沙箱 |
| MOVO Enterprise Layer | Agent 怎么进入企业环境被多人、安全、可控地使用 | 组织、用户、知识、模型、Skill、MCP、审计、Token 治理 |

**为什么企业不直接在 DSH 上自己搭一层？** 封装 DSH 不难，长期维护企业层才是真正的成本：DSH 升级要不要跟、现有集成会不会受影响、身份/组织/权限/知识边界长期维护、Plugin 准入/权限/审计机制、会话/事件/监控/故障恢复、Agent 统一创建发布运营入口。这些单看都不复杂，进入长期运行阶段就变成持续的平台工程成本。

## 核心功能（社区版已有）

| 能力 | 说明 |
|------|------|
| Agent 对话与工作空间 | 统一工作空间对话、调 Skill、用工具、完成任务 |
| 模型管理 | 统一接入管理多家大模型；后续做模型权限、配额、成本治理 |
| 企业 Skill | Skill 安装/管理/企业内共享，支持 ZIP Skill、专家包；持续补兼容能力 |
| Tools 与 MCP | 统一工具管理入口；后续延伸到工具权限、审批、审计、敏感操作控制 |
| 企业知识与 RAG | 知识文档管理 + RAG，把企业文档知识给 Agent 用 |
| 组织与角色 | 企业组织、用户、岗位角色管理——资源权限控制的基础 |
| Token 和使用统计 | 管理员了解平台调用与成本 |
| 系统审计 | 谁做了什么、何时发生、可追溯 |

## 关键洞察（文中三个判断）

1. **Skill 开始变成企业资产**——未来企业用 Agent 真正沉淀的不只是 Prompt，而是可复用的 Skill。资深销售的"拿到客户材料先分析什么→查什么数据→按什么格式出报告"这类经验，过去存在人脑里，沉淀成 Skill 后，Agent 平台最终沉淀的是**企业经验资产**。Skill 具备安装、共享、权限、管理能力后，就从个人 Prompt 变成企业软件的正式能力单元。
2. **私有化解决的不是"能不能部署"，而是"企业敢不敢真正用起来"**——企业敢不敢全量用 Agent，取决于数据是否可控：知识留内网、客户数据不外泄、内部账号安全、CRM/ERP 受控调用、日志完整留存。这些没解决，企业只敢小范围试用。
3. **Runtime 与 Enterprise Agent Platform 会逐渐分层**——Runtime 负责执行，Enterprise Layer 负责把它变成可以在企业中长期运行的产品。

## 安装与部署

```bash
git clone https://github.com/himovo/movo.git
cd movo && chmod +x movo && ./movo up
# 打开 http://localhost:3000/admin/setup
```

- GitHub：https://github.com/himovo/movo （42⭐ / 7 forks，2026-09-01 开源，v0.1.14，活跃开发中，单一公司 himovo 主导）
- 官网：https://www.himovo.com ｜ 文档：https://www.himovo.com/guide/admin-overview.html
- 要求：Git + Docker Desktop（或 Docker Engine + Compose v2）、8GB 内存、20GB 磁盘、至少一个模型 API 凭证
- Docker Compose 默认启动 12 个服务：网关、2 个 Web 应用、3 个应用 API、DSH Runtime Host、文档 worker、MongoDB、Redis、Weaviate、一次性密钥引导服务
- Browser Agent / Code Agent 需另装 **MOVO Desktop**（专有软件，闭源，不在社区版源码内）

## 技术栈（来自仓库）

Python 72.1% / Vue 17.8% / TypeScript 7.4%。架构：用户工作空间与管理台 → Gateway → Chat API + Admin API → DSH Runtime Host（Node.js）+ 文档服务 → MongoDB/Redis/Weaviate。仓库结构：`apps/user-web`（Vue3 用户端）、`apps/admin-web`（Vue3 管理台）、`services/chat-api`（FastAPI 会话/任务/Agent/Skill/DSH 网关）、`services/admin-api`（FastAPI 组织/用户/模型管理）、`services/document-parser`（文档解析/预览/检索 API 与 worker）。

## 战略分析

**1. 与知识库"生产级 Agent 安全"谱系的对照**

| 维度 | MOVO | OpenBot | AgentDesk | BrowserOS |
|---|---|---|---|---|
| 定位 | 企业 Agent 平台（DSH 之上） | 每 Agent 独立虚拟电脑 | 任务卡多 Agent 开发流 | 带登录态本地 AI 浏览器 |
| 隔离 | 多租户/组织/角色权限 | 容器 + gVisor | 工作区/路径白名单 | 本地双浏览器 |
| 审计 | 系统审计 + Token 统计 | PostgreSQL 前置写日志 | Git 事件/outbox | 操作录像 |
| 治理 | 模型/Skill/工具准入 | CEL 策略 Fail-Closed | 30 条不变量 | - |
| 开源程度 | ⚠️ Community License（非 OSI） | MIT | MIT | 开源 |

**2. 项目成熟度判断（42⭐ 需正视）**：按热门项目筛选标准，MOVO 目前星数很低（42⭐、7 forks、单贡献者、开源仅两周），处于极早期——文章是厂商视角的产品发布文，实质是软广。但它的**架构思路值得借鉴**：DSH Runtime + Enterprise Layer 分层，与 Hermes Agent（Runtime + 技能库 + TencentDB Memory + 各平台 Channel）是同构命题——Hermes 解决"Agent 怎么运行"，多用户/组织权限/配额/审计这类企业层是不同的问题域。

**3. 对现有工具链的启发**：
- **Skill = 企业经验资产** 的定位与知识库积累的 90+ skill 方法论同向——小谭在废才俱乐部学的"每期方案的技能/规则/配置"正是这个路线的个人版；MOVO 把它做成多用户可共享、可治理的平台能力
- **Skill 治理三板斧**（准入/权限/审计）与 AgentDesk 的 30 条不变量、OpenBot 的 CEL 策略引擎是同一问题在不同层的解法
- 若 InStreet/业务场景要做多用户 Agent 服务，MOVO 是"DSH 系"可参考的开源样板，但 42⭐ + Community License 限制（禁 SaaS 化、禁 OEM 白标）意味着只适合借鉴架构，不适合直接押注

## 关联阅读

- [[2026-09-02-DeepSeek-Harness缓存优化-源码拆解]] — 同为 DSH 生态，成本视角 vs 企业平台视角
- [[2026-09-04-OpenBot-每Agent独立电脑安全网关]] — 安全治理对照：容器隔离+CEL 策略 vs 企业权限+审计
- [[2026-08-18-Yao-Meta-Skill-Skill-OS-技能工厂与治理系统]] — Skill 治理的元层面解法（Skill IR/门禁/反模式库）
- [[2026-08-18-Buzz-多Agent协作平台实操]] — 多 Agent 多模型协作的另一种组织形态
- [[2026-08-18-轻量化业务应用-四步渐进式验证框架]] — 企业落地的渐进式验证方法论

## 归档日志

- 2026-09-16 归档（Firecrawl 抓取公众号原文 + GitHub 仓库实时数据）
