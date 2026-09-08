---
title: "Professor Synapse：给 AI 装个'专家调度系统'的开源 Skill"
source: "patrick（Ai实用talk）"
source_url: "https://mp.weixin.qq.com/s/1nxGBUV_jd_w2KIZIcbFBw"
date: "2026-08-25"
tags: [Professor-Synapse, Skill, 专家调度, 多Agent, 记忆, 开源]
---

# Professor Synapse：把"角色扮演"做成"系统工程"的专家调度 Skill

> **一句话**：3.4k star 开源 Skill（Synaptic Labs），给 AI 装了个"专家调度系统"——说一句 "help me with..." 就召唤领域专家接管对话，越用越懂你：专家库自生长、持久记忆、模式学习、多 Agent 辩论。

## 项目信息

- **仓库**：profsynapse/professor-synapse（3.4k⭐）
- **触发**："Help me with my startup go-to-market plan" / "I need guidance on database sharding strategy"
- **机制**：先评估问题复杂度 → 从"专家名册"找匹配专家接管对话

**与角色扮演的本质区别**：角色扮演靠提示词里一段人设描述，换个话题就破功；Professor Synapse 靠一整套机制把"专家"变成持续积累的资产。

## 四大核心机制

### 01 专家库（自生长）
- 匹配不到合适的专家？内置"领域研究员"去网上调研该领域
- 用标准模板创建新专家，存档进专家库
- 下次类似问题直接命中——**用得越多，专家库越厚**

### 02 持久记忆
- 每个专家说过的话、用户偏好、哪些方案有效，打标签存进记忆库
- 下次会话捞出相关记忆，不每次都从零开始

### 03 模式学习
- 哪些做法好用、哪些是反模式，自己记下来
- 这次踩的坑，下次不踩

### 04 多 Agent 辩论
- 遇到怎么选都有道理的复杂决策，召集多个专家从不同角度辩论一轮
- 综合各方意见给建议——"不像工具，更像一个团队"

## 更新不丢数据

- 自建专家、积累的记忆在更新时都不丢
- 自动从 GitHub 拉新版本，只更新核心部分，用户资产原样保留

## 三种安装方式

1. **Universal Prompt 版**：复制 Prompt 进任意 AI 系统提示（ChatGPT/Claude/Gemini 通用）
2. **Claude Code 插件**：
   ```
   /plugin marketplace add profsynapse/professor-synapse
   /plugin install professor-synapse@professor-synapse
   ```
3. **Codex Portable 版**：文件夹丢进 skills 目录

**验证**：`python3 scripts/summon.py --list` 查看名册

## 评价与局限

**亮点：** 把"角色"做成可生长、可记忆、可辩论的系统——对比 Claude Code 官方 subagents（功能重叠但无此系统性）

**局限：**
- 需要带具体问题来，没任务时只是人设空转
- 文档/社区基本英文，中文资料少
- **项目没标 License**——商用/二次分发需注意

> 金句："AI 时代，专家不再靠背书，靠调度——谁能把对的专家叫到对的场景里，谁就拥有了'专家'。以后 AI 的常态不是'一个 AI 什么都会'，而是'一个 AI 调度一群专家'。"

## 战略分析

**1. 与 Hermes skill 路由的关系（同类不同形态）：**

| 维度 | Professor Synapse | Hermes skills_list | Yao Meta Skill | gbrain |
|---|---|---|---|---|
| 调度 | 专家名册 + 复杂度评估 | description 触发匹配 | Skill IR trigger_surface | 语义检索 |
| 自生长 | 领域研究员建新专家 | 手动 skill 创建 | SkillOS 创建流程 | - |
| 记忆 | 标签化持久记忆 | 无（会话级） | - | 知识库 |
| 辩论 | 多专家辩论 | delegate_task 子代理 | 咨询圆桌 | - |

Professor Synapse 的独特点：**专家作为一等公民 + 动态创建 + 跨会话记忆**——Hermes 的 skill 是静态的（需要时手动建），Professor Synapse 的专家是"不够就现场调研生成"。这是"skill 动态化"的方向。

**2. 与最近归档的呼应：**
- **多 Agent 辩论** = AgentDesk 咨询圆桌（0-3 席独立意见）+ Buzz 头脑风暴（不同模型视角碰撞）的 Skill 化——"复杂决策召集多方辩论"已是多来源验证的共识模式
- **持久记忆标签化** = TencentDB Memory / gbrain 的记忆分层思路在 skill 层的应用
- **更新不丢用户资产** = 版本隔离意识（与 AgentDesk 的 AGENTDESK.lock.json、Yao 的 release-manifest 同哲学：核心更新不破坏用户数据）
- **领域研究员动态建专家** = "轻量化业务应用"里 AI 辅助建模的思路

**3. 落地建议：** 对本机 Hermes 的价值主要在理念借鉴：①"专家调度"可作为新 skill 的设计模式（按领域分专家 + 复杂度路由）；②注意 License 未标——不商用、个人试用没问题；③中文资料少，用 Universal Prompt 版体验成本最低。

## 关联阅读

- [[2026-08-18-AgentDesk-任务卡驱动多Agent开发工作流]] — 咨询圆桌 = 多 Agent 辩论同构
- [[2026-08-18-Buzz-多Agent协作平台实操]] — 头脑风暴 = 多视角碰撞
- [[2026-08-18-Yao-Meta-Skill-Skill-OS-技能工厂与治理系统]] — Skill 治理与动态创建
- [[2026-07-22-GBrain-AI-Agent记忆系统深度解析]] — 持久记忆分层

## 归档日志

- 2026-09-04 归档（Firecrawl 成功；封面已存 assets）
