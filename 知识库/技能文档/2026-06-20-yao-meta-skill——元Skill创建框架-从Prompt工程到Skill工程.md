# yao-meta-skill — 元 Skill 创建框架：从 Prompt 工程到 Skill 工程

> **来源：** 公众号「开源星探」
> **发布时间：** 2026-06-20
> **原文：** https://mp.weixin.qq.com/s/5kM-fJcTwHiZCPsgC6v_6A
> **GitHub：** https://github.com/yaojingang/yao-meta-skill
> **安装：** `npx skills add https://github.com/yaojingang/yao-meta-skill --skill yao-meta-skill`

## 概述

yao-meta-skill 是一个**元 Skill**——即「创建 Skill 的 Skill」。它将 Skill 的构建从"写一段 Prompt"提升到**工程化生命周期**管理：从意图对话到路由设计、质量评估、跨平台打包、版本治理，形成完整闭环。

核心理念：**Build reusable skill packages, not long prompts.**

## 技术栈与定位

| 维度 | 内容 |
|------|------|
| 作者 | yaojingang |
| 定位 | Skill 的脚手架 + CI/CD 二合一 |
| 安装方式 | npx skills add 或 git clone |
| 输出目标 | OpenAI / Claude / 通用 三种格式 |
| 可移植性评分 | 100/100（自述）|
| 适用阶段 | 个人探索 → 团队复用 → 基础设施 |

## 完整工作流

```
原始输入（工作流笔记/提示词/对话记录/文档）
  → 意图对话 → reports/intent-dialogue.md
  → 路由设计 → SKILL.md + agents/interface.yaml
  → 参考扫描 → reports/reference-synthesis.md
  → 路由评估 → evals/trigger_cases.json
  → 报告生成 → reports/skill-overview.html
  → 打包分发 → 跨平台兼容的成品包
```

## 三大运行模式

| 模式 | 适用场景 | 流程重量 |
|------|---------|---------|
| **Scaffold**（脚手架） | 个人探索、快速原型 | 最轻，只保留必要结构 |
| **Production**（生产） | 团队复用 | 加入质量门槛和评估流程 |
| **Library**（库） | 共享基础设施、元技能 | 完整的治理和文档 |

原则：**流程的重量应与风险成正比。**

## 质量评估体系

这是 yao-meta-skill 最突出的工程化特性——Skill 不再靠"感觉"判断好坏，而是可量化评估：

| 评估项 | 脚本 | 说明 |
|--------|------|------|
| 触发词评估 | `trigger_eval.py` | train/dev/holdout 三层测试集验证路由准确率 |
| 描述优化盲评 | `run_description_optimization_suite.py` | 多版本 description 盲评对比 |
| 法官式盲评 | `judge_blind_eval.py` | 独立评分模型二次验证 |
| 资源边界检查 | `resource_boundary_check.py` | 确保不越权操作 |
| 治理检查 | `governance_check.py` | 验证元数据完整性 |
| 路由混淆检测 | `route_scorecard.md` | 防止相似 Skill 相互干扰 |

## 典型目录结构

```
my-skill/
├── SKILL.md                  # 入口路由文件（核心）
├── agents/
│   └── interface.yaml        # Agent 接口声明
├── references/               # 参考材料
├── scripts/                  # 确定性脚本
├── evals/
│   ├── trigger_cases.json
│   └── blind_holdout/
├── reports/
│   ├── intent-dialogue.md
│   ├── skill-overview.html
│   ├── iteration-ledger.md
│   └── promotion-decisions.md
├── manifest.json             # 治理元数据（生产级别）
└── VERSION                   # 版本号
```

按需添加，不强制所有目录。

## 战略分析

### 与 ANYTHING2SKILL 的对比

两者定位互补而非重复：

| 维度 | ANYTHING2SKILL | yao-meta-skill |
|------|---------------|----------------|
| 核心能力 | 知识→Skill 的编译 | Skill→工程化的生命周期管理 |
| 输入 | 文档/笔记/知识 | 已有的 prompt/工作流/需求描述 |
| 输出 | SKILL.md | 完整的工程目录+评估报告 |
| 质量 | 靠用户审查 | 内建评估体系（盲评/对抗/路由测试）|
| 治理 | 无 | 迭代账本、成熟度评分、升级决策 |

### 与 Hermes Skills 的关联

Hermes 已有完整的 skills 系统（`skill_manage`、`skill_view`、`skills_list`），但缺少：

1. **评估体系** — 目前靠人肉判断技能好坏，yao-meta-skill 的 trigger_eval + 盲评机制可以引入
2. **路由检测** — 多 skill 之间的路由冲突检测，当前是隐式的
3. **治理记录** — 迭代账本 + 成熟度评分，可用于 skill 版本管理
4. **跨平台打包** — Hermes skills 目前是 Hermes 专属格式，yao-meta-skill 支持 OpenAI/Claude/通用三格式

### 关键方法论资产

项目的 `docs/`、`references/`、`failures/`（反模式库）中沉淀了可操作的方法论：

- 意图对话手册：2-3 个高杠杆问题快速锁定需求
- 参考扫描策略：外部标杆 → 用户偏好 → 本地适配
- 原型选择指南：判断是否应该做成 Skill
- 非 Skill 决策树：识别"这就是一次性任务，别过度工程化"

## 相关文档

- [[技能文档/2026-06-11-ANYTHING2SKILL：从知识编译技能，帮Agent把知识变成能力]]
- [[技能文档/2026-06-01-AI-Agent-Skill-工程化-01：写-Skill-不是写-Prompt，构建可迭代的工程体系]]
- [[技能文档/Claude Prime 一键告别重复 Prompt 工程]]
- [[技能文档/2026-06-04-SkillDAG把孤立技能变成会进化的关系图]]
