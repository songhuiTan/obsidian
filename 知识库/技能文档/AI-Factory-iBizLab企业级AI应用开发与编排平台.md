---
source: https://mp.weixin.qq.com/s/J5pH0fJslm3DCgMsLcRxfQ
author: 一飞开源
date: 2026-06-25 10:10
archived: 2026-06-25
tags: [AI Factory, iBizLab, 企业级AI, 多智能体, 零代码, 云边协同, MIT协议, 企业级平台, 归档]
---

# AI Factory — 企业级 AI 应用开发与编排平台

> **核心摘要：** iBizLab 开源实验室打造的新一代企业级 AI 应用开发与编排平台（MIT 协议）。零代码/低代码构建智能体，Hub-Agent → Sub-Agent → Skill → Kernel Tools 四层分级架构，支持云边协同、知识库深度绑定、沙盒隔离。与 RAGFlow/Dify、Hermes-Agent/OpenClaw 形成差异化定位——专注政企关键业务场景的可控、可复现、可审计。

---

## 项目定位

- **项目名：** AI Factory（iBiz AI Factory）
- **开源协议：** MIT
- **开发者：** iBizLab 开源实验室（基于 iBizModeling 打造）
- **来源：** https://gitee.com/ibizlab/aifactory
- **定位：** 面向政企关键业务场景的"智能体操作系统"，强调**有限自主编排、分级调度、可控可复现**

## 设计理念

- **有限自主编排** — 在明确边界和权限下执行任务，避免底层错误决策
- **分级调度架构** — Hub-Agent → Sub-Agent → Skill → Kernel Tools，严格分层
- **Skill 固化与定制化** — 核心技能通过 SkillHub 标准化，同时支持定制化扩展
- **工具最小化** — 底层 Kernel Tools 只保留基础原子操作
- **政企落地优先** — 云边协同、沙盒隔离、低算力+短上下文环境适配
- **创造力最大化** — 在可控和安全前提下压榨 AI 业务价值

## 核心架构

### 1️⃣ 角色化总线（Hub-Agent）
"多专家调度官"，负责组织/团队/场景的全局控制：
- 多 Hub 实例隔离（按组织/业务领域）
- 挂载特定角色 Prompt + **支配清单**（可调度的 Sub-Agent 与 Skill 范围）
- 负责复杂需求拆解分发

### 2️⃣ 原子智能体（Sub-Agent/Standalone）
"专项执行官"：
- **从属模式（Sub-Agent）**：由 Hub 激活，运行于临时沙盒，用完即销毁
- **独立模式（Standalone）**：独立 URL/Token，支持 Cron Job 或 Pipeline 直接驱动
- **能力配给制**：每个 Agent 仅支配清单内的 Skill，实现最小权限控制（PoLP）
- **模型级联**：每个 Agent 独立指定模型，实现"高低快慢"算力搭配

### 3️⃣ 能力层 — 解耦化技能总线（Skill Bus）
- **Standard Skills（Global SkillHub）**：监听外部 Git 仓库，热加载同步
- **Web-Defined Skills（Local Extension）**：管理后台 UI 快速录入的业务规约
- 所有技能遵循标准 **Input/Output Schema**，支持技能级联（Chaining）

### 4️⃣ 基础层 — 原子工具集（Kernel Tools）
不可随意更改的原子指令集：
- `execute_bash` / `execute_cloud`（RPC/REST 调用）
- `read_file` / `write_file` / `apply_patch`
- `fetch_kbs` / `fetch_kb_chunks`（知识库路由与向量检索）
- `sub_agent`（递归编排，派生逻辑隔离的任务沙盒）

## 核心特性

| 维度 | 实现方式 | 目的 |
|------|---------|------|
| 协同性 | Cloud-Edge Skill Runner | 云边协同，打破内网数据隔离 |
| 知识性 | Deep KB Binding | 高精度召回与知识回填 |
| 可追溯性 | Anchor & Portal | 从 AI 结论到原始证据一键回溯 |
| 灵活性 | Multi-Hub / Standalone | 覆盖人机问答与无人值守全场景 |
| 隔离性 | Context Sandbox | 子任务沙盒运行，用完即焚 |

### 云边协同（Cloud-Edge Skill Runner）
- **云端大脑**：高维思考、多 Agent 编排、跨知识库决策
- **边端触手**（Skill Runner）：本地执行敏感 Skill 代码、访问本地文件、驱动局域网程序
- 通过高可靠异步消息通信

### 知识库深度绑定
- Agent 与知识库"空间绑定"（声明式知识边界）
- 跨库动态路由（按意图自动切换/联合检索）
- 双向同步（分析结果反向写回知识库）
- **Anchor & Portal**：从 AI 结论到原始物理文件的一键回溯

### 上下文沙盒隔离
- 子任务分配独立上下文堆栈（Context Stack）
- **用完即焚（Ephemeral Instance）**：结论返回后沙盒实例立即销毁
- 主模型仅记录结构化结论，保持高信噪比

## 场景功能

1. **企业级知识库与权限隔离** — 组织-团队-个人网格架构，物理与逻辑权限隔离
2. **跨知识库智能检索辅助对话** — 多知识库动态路由，数据穿梭门一键回溯
3. **多角色中控总线（Hub）** — 动态调配 Sub-Agent 与技能清单
4. **零门槛即时新建专属智能体** — 自然语言描述，在线秒级编译绑定
5. **用户专属 OpenAI 接口** — 兼容标准接口，Token 绑定身份与权限
6. **对话式报表** — AI 自动写 Python 代码分析 Excel/CSV，沙箱内编译运行
7. **记忆串联** — 特征提取/类案推送/规则推理技能级联
8. **深度研究** — 规划→执行→总结 多 Agent 异步协同
9. **云边协同** — 大批量数据分批解析，Sub-Agent 并发沙盒处理
10. **标准化工作流 + 场景核查套件** — 确定性 Workflow + 可插拔场景套件 + Cron Job

## 技术选型

- 要求：CPU ≥ 4 核，RAM ≥ 16 GB，Disk ≥ 50 GB，Docker ≥ 24.0.0
- 部署方式：`docker-compose up -d`
- 源码：https://gitee.com/ibizlab/aifactory

---

## 与主流产品的对比

| 对比维度 | RAGFlow/Dify（早期 RAG 时代） | Hermes-Agent / OpenClaw（高度自主新时代） | AI Factory（企业级智能体操作系统） |
|---------|------------------------------|----------------------------------------|--------------------------------|
| **设计理念** | 以 RAG 和工作流为核心 | 以 LLM 自主工具调用为核心 | 稳定且安全的有限自主 AI 执行 |
| **架构范式** | 硬耦合：LLM+RAG+工作流+插件 | 扁平化自主 | **分层分级**：Hub→Sub→Skill→Tools |
| **AI 自主能力** | 受限（工作流/Prompt 约束） | 高度自主（规划/选工具/执行） | **有限自主（受控编排）** |
| **可控性与安全** | 中等 | 弱（自由度高，难以预测） | **强**（最小权限+沙盒+可追溯） |
| **复现与稳定性** | 中等 | 弱（结果不可预测） | **强**（分级编排+稳态执行） |
| **适用场景** | 通用问答/客服/内容生成 | 研究探索/创新场景 | **政企关键业务场景** |
| **上下文隔离** | 无 | 部分隔离 | **沙盒快照+用完即焚** |
| **任务调度** | API 触发 | 简单队列/脚本 | **Cron Job + 场景化调度** |
