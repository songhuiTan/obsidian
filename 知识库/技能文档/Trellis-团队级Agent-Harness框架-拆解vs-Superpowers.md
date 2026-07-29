# 还只用 Superpowers 吗？别错过这个黑马项目！Trellis 团队级 Agent Harness 框架拆解

> **来源**：微信公众号「AI海森」(AI沐)
> **日期**：2026-07-07
> **链接**：https://mp.weixin.qq.com/s/qG7SYIc2IQ2UbhUub2Hwcg
> **标签**：`Trellis` `Superpowers` `Agent Harness` `团队协作` `LLM Wiki` `工作流框架`
> **系列**：AI 海森 Trellis 系列（第3篇）

---

## Trellis 是什么

Trellis（GitHub 12k⭐）定位：**Team-level Agent Harness with built-in LLM Wiki**

三层架构：

```
┌─ Agent Harness（执行骨架） ───────────────────────┐
│  管理 workflow 状态、hook、skill、子代理调度         │
├─ 内置 LLM Wiki（知识库） ──────────────────────────┐
│  Spec（团队规范）+ Task（任务知识）+ Journal（会话记忆）│
│  以文件形式存入仓库                                 │
├─ 团队层（协作层） ──────────────────────────────────┐
│  文件全部 git 版本化，团队共享，适配 16 个 AI 编程平台 │
└───────────────────────────────────────────────────┘
```

**解决的问题**：
- **项目失忆症**：上下文一压缩，规范全忘。Trellis 用"文件即记忆"绕开——不靠模型记住，靠仓库文件重新加载。
- **团队规范不统一**：每个人喂给 AI 的规则不同。Spec 变 git 版本化共享资产。
- **多工具协作割裂**：.trellis/ 核心结构跨 16 个平台复用，换工具不用重搭。

---

## Trellis vs Superpowers 对比

| 维度 | Trellis | Superpowers |
|------|---------|-------------|
| **核心定位** | Team-level Agent Harness + 内置 LLM Wiki（团队级基础设施） | 面向单会话的软件工程方法论（Skill 组成的开发 SOP） |
| **核心问题** | 跨会话/跨任务/跨团队的"项目失忆症"与规范不统一 | 单次任务内的上下文膨胀与"AI 跳过设计直接写代码" |
| **核心抽象** | Spec（规范）+ Task（任务）+ Workspace Journal（会话记忆） | Skill（YAML+Markdown 描述的可组合技能） |
| **上下文加载** | JSONL manifest 精确点名所需文件，固定顺序注入 | 关键词触发对应 Skill，偶发"未触发就直接写" |
| **记忆持续性** | 任务状态标记 + 按开发者区分的持久 journal，跨会话续接 | 全新上下文子代理，仅解决单任务内上下文污染 |
| **团队协作** | Spec 库 git 版本化共享，`update-spec` 沉淀为全团队规则 | 个人技能覆盖机制，非团队级设计 |
| **工作流程** | 三阶段：Plan → Execute → Finish | 七阶段：Brainstorming → Worktree → Plan → Subagent Dev → TDD → Review → Finish |
| **审查机制** | `trellis-check` 子代理自查自修+重跑验证 | 两阶段：Spec Review + Code Quality Review |
| **跨工具适配** | 核心结构不变，自动生成16个平台适配层 | 每换宿主环境单独安装，部分能力依赖平台开关 |
| **适用场景** | 多人团队、长期项目、混合工具栈 | 需要长期迭代的项目，不适合快速原型 |

### 两者关系

**不是竞品，而是解决问题维度不同：**
- Superpowers = 坐在你旁边的技术 Leader，把软件工程最佳实践焊进单次开发流程
- Trellis = 团队级基础设施，解决跨会话、跨任务、跨团队的问题

---

## 核心区别详解

### ① 上下文加载：结构化清单 vs 关键词触发
Trellis 用 JSONL manifest 精确点名"这个任务要读哪几个 Spec"，按固定顺序注入。Superpowers 依赖关键词触发，实测经常出现"没触发任何技能就开始写代码"。

### ② 记忆持续性：跨会话状态机 vs 单任务隔离
Trellis 给每个任务状态标记（`planning → in_progress → 归档`），加上按开发者分的 journal，可无限期跨会话续接。Superpowers 没有为"第二天打开项目/换人接手"设计持久状态。

### ③ 团队规范沉淀：git 版本化 Spec vs 个人覆盖
一个人踩坑通过 `/trellis:update-spec` 沉淀进 Spec，全团队自动读到。Superpowers 的覆盖机制面向个人定制。

### ④ 跨工具适配：核心不变只生成适配层
`.trellis/` 目录完全不变，`trellis init` 只生成对应平台的 `.claude/` / `.codex/` / `.cursor/` 适配文件。

---

## 官方 FAQ 精选

### 会话历史存在哪？
```
.trellis/workspace/
├── index.md              # 所有开发者的主索引
└── {your-name}/
    ├── index.md          # 个人索引
    └── journal-N.md      # 会话日志（约每2000行新起一个文件）
```

### 大仓老代码会不会塞满上下文？
不会。每个 spec 只覆盖一个主题，文件本身小。brainstorm 时 AI 只把当前任务相关的 spec 路径写进 `implement.jsonl`，hook 只注入清单里列出的文件。主会话只读 spec 的 index（仅路径）。

### 多 task 同时跑会污染吗？
每个 task 有独立目录 `.trellis/tasks/<MM-DD-slug>/`，sub-agent 只读当前 active task 的 jsonl。开发者级 journal 在 `workspace/<name>/` 也是隔离的。

### 能否和 Superpowers 等框架共用？
❌ **不推荐**。本质都是工作流框架，各有注入机制和阶段定义。两套在同一个会话跑会导致：AI 同时收到两套阶段提示输出不可预期、hook 互相覆盖上下文膨胀、调试成本剧增。**每个会话只用一套工作流框架。**

### 有 TDD 支持吗？
目前还没有内置 TDD 工作流（默认 implement → check 事后检查）。TDD 是 0.5.x roadmap 项。在此之前可把 TDD 要求写到 `spec/testing.md`。

---

## 迁移流程

1. `npm install -g @mindfoldhq/trellis@latest`
2. 项目根下 `trellis init -u your-name`
3. 打开 AI 会话，brainstorm skill 帮你填初始 spec
4. 按团队约定补充核心 spec
5. commit `.trellis/` + 对应 `.{platform}/`
6. 队友 pull 后各自 `trellis init -u their-name`

---

## 总结

Trellis 想解决的不是"让 AI 更聪明"，而是**让 AI 记得住、守得住规矩**。把团队协作、知识沉淀、规范执行从一次性提示词变成可版本化、可迭代、可传承的基础设施。

> 如果你或团队正被"AI 写代码总是失忆、规范总是不统一"折磨，这可能是目前把这个问题想得最透彻的一套方案。

---

**官方文档：** https://docs.trytrellis.app/zh/
**GitHub：** https://github.com/mindfold-ai/Trellis
