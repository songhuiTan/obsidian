---
title: "我用肥波搓了一个 Lisp 写的 Agent OS，比我用 Rust 自己搓的还强大"
source: "小张（老码小张）"
source_url: "https://mp.weixin.qq.com/s/fOg6c-p6kIJQF_-GshcphA"
date: "2026-07-18"
tags: [AI, Agent, Lisp, Clojure, Rust, Agent OS, Event Sourcing, Hermes, 开源]
---

## 事件背景

作者小张用一年多时间用 Rust 搓了 **Small Rust Hermes**（27000 行、14 个 crate、CLI + Tauri 桌面 + 微信桥 + 飞书桥 + Flutter 移动端），然后上周用"肥波"（Claude Code）搓了一个新的 Lisp Agent OS（Clojure/JVM 实现），评价是——"比我的 Rust Hermes 还强大，强大的层次不一样。"

核心判断转变：**Agent 不是 LLM 的延伸，Agent 是一个完整的运行时系统。**

> 2025 年的认知：Agent = LLM + Prompt + 几个 Tool（一个对话循环）
> 2026 年的认知：Agent = Runtime + Plan + Event Store + Policy Engine + Tool Broker + Sandbox + Skill Runtime + Replay（一个操作系统）

## 核心观点：LISP 为什么适合 Agent OS

> "Rust 强在：零成本抽象、内存安全、确定性延迟——适合引擎、嵌入式、数据库。
> Clojure 强在：抽象、数据流转、可观察性、可恢复、可审计——需要 homoiconicity、REPL、不可变、宏、DSL。"

LISP 哲学在 Agent 场景是天然的：
```
S-Expression = 数据 = 代码 = Plan = Workflow = Event
    ↑ 全是同一个东西
```

Rust 要表达同样的东西，得用 enum + struct + serde + macro 写三层。Clojure 原生就是 S-Expression。

## 四个核心能力

### 一、计划可见（Plan as S-Expression）

市面所有 Coding Agent 都是黑盒——输入、转圈、输出 diff，中间看不见。Lisp Agent OS 要求 Agent 在执行前先生成一个结构化 Plan，计划改版不能覆盖，必须出新版本：

```
plan/version 1 → 执行中发现新问题 → plan/version 2（记录原因"测试失败，增加依赖检查步骤"）
```

你能看到 Plan 的演化历史，理解 Agent 为什么改变方向。

### 二、技能可编程（AGENTS.md + SKILL.md + workflow.edn）

**三层渐进**：

```
AGENTS.md   → Markdown 文档：告诉模型"如何思考"（项目规范、上下文）
SKILL.md    → Markdown + YAML frontmatter：告诉模型"何时用什么技能"
workflow.edn → S-Expression 数据：告诉系统"如何执行"（受验证、受约束）
```

**workflow.edn 是数据，不是代码。** 关键约束——六道闸：

1. **节点白名单**：只能用 10 个（workflow/sequence/parallel/tool/model/approval/if/retry/emit/return/stop）
2. **表达式白名单**：只能用 9 个（get/eq/not/and/or/contains/success?/failed?/empty?）
3. ❌ 禁止：eval、任意 Clojure 函数调用、Java Interop、宏展开、自定义 Reader Tag、无限递归、直接文件/网络访问

> "很多人怕 LISP 是因为' eval 太危险'——这个项目根本不让模型碰到 eval。"

### 三、经验可沉淀（Event Sourcing）

所有 Agent 状态变化都必须形成事件，禁止只改内存而不记录。每个事件结构：

```
{:event/id           #uuid "..."
 :event/type         :tool/completed
 :event/actor        :tool-broker
 :event/correlation-id #uuid "..."  ; 因链
 :event/causation-id   #uuid "..."  ; 果链
 :event/payload      {...}
 :event/hash         "sha256:..."}
```

30+ 种事件类型（session/created、plan/updated、tool/completed、approval/resolved……）

**状态机是事件的纯函数投影** — `(transition current-state event) => next-state`

状态恢复 = 读最近 Snapshot + 重放之后的事件。每 20 个事件/每次 Plan 更新/每次审批前自动 Snapshot。

> "Rust Hermes 没有这个——它靠 Markdown 文件作为状态，崩了你能恢复'记忆'，但恢复不了'运行时状态'。"

### 四、运行可回放（Replay）

`laos replay <run-id>` 进入只读回放模式：

- ←/→：上一个/下一个事件
- Space：自动播放/暂停
- P/S/T/M/D/E：查看当时计划/状态/ToolCall/模型响应/Diff/错误

不调用模型、不调用工具、不修改 Workspace。

> "市面 99% 的 Agent 都没有这个。"

## Rust Hermes vs Lisp Agent OS 的维度对比

| 维度 | Small Rust Hermes | Lisp Agent OS |
|------|------------------|---------------|
| 形态 | Agent App（产品） | Agent Runtime（操作系统） |
| 主打 | 多端、自进化、移动 | 可观察、可控制、可恢复、可审计 |
| 类比 | iOS | Linux 内核 |
| Plan | ReAct Loop，无显式 Plan | 版本化 S-Expression Plan |
| 状态持久化 | Markdown 文件 | Event Sourcing + Snapshot |
| 回放 | ❌ | ✅ TUI 回放 |
| 自我进化 | ✅ Reflect Engine | ❌（第一版被动记录） |
| 冷启动 | < 2 秒 / 15MB | 2-3 秒 / 200MB+ |

核心认知转变：**Agent App 的天花板是 Runtime 的天花板。Agent OS 才是真正的护城河。**

## 战略分析

**与 Hermes Agent 体系的直接相关度：极高。**

这篇文章直接触及 Hermes Agent 的核心设计问题。作者做的"Small Rust Hermes"虽然名称不同，但设计目标与 Hermes Agent 高度重叠（本地 Agent、多端支持、自我进化）。他的认知转变——从"Agent 是 LLM 延伸"到"Agent 是完整操作系统"——正是 Hermes 当前面临的架构决策。

**最值得借鉴的四个设计：**

1. **workflow.edn 数据化工作流 + 六道闸安全约束** — 这是对 Hermes skill 系统的直接进化。Hermes 的 skill 用 SKILL.md（YAML frontmatter + markdown body）编写，但执行逻辑在 skill 内部自包含。workflow.edn 把工作流从代码升级为数据，用白名单约束确保模型不能"逃逸"。**这与 HarnessX 的 Processor Hook 点体系呼应**——两个项目都在追求"可组合、受约束的执行流"。

2. **Event Sourcing 全套实现** — 30+ 事件类型、correlation/causation 链、Snapshot 恢复。Hermes 当前有 TencentDB Memory 做对话记忆，但没有运行时的完整事件溯源。引入事件溯源可以让 Hermes 实现"运行可回放"——对调试复杂 Agent 行为至关重要。

3. **版本化 Plan** — Agent 在执行前生成结构化 Plan，修改必须出新版本。Hermes 的子代理当前是 ReAct-like：走一步看一步。版本化 Plan 让 Agent 的决策历史可审计、可回滚。

4. **回放 TUI** — 像视频播放器一样的 Agent 运行回放。虽然 Hermes 的子代理已经可以记录执行轨迹，但缺少一个系统的回放界面来逐事件查看。

**差距与局限性：**

1. Clojure/JVM 冷启动 2-3 秒，运行时 200MB+——不适合 Hermes 的 CLI 即时响应场景
2. 第一版没有自进化机制，被动记录 vs Hermes 的主动学习
3. 项目在微信群内分享源码，未完全公开
4. 只有 Coding Agent 场景，不如 Hermes 的通用性

**整合可能性：中高。** 最推荐的整合路径是从 Event Sourcing 和 workflow.edn 入手：
- Event Sourcing 可以逐步叠加到 Hermes 的运行时层，不破坏现有 skill 系统
- workflow.edn 的白名单 DSL 思想可以引入 Hermes skill 的"安全模式"
- 版本化 Plan 可以作为子代理执行的可选模式（plan-first vs react-first）

## 相关资源

- GitHub 项目：https://github.com/coder-brzhang/list-agent（应为 lisp-agent）
- 作者微信群小范围分享源码与设计文档

## 归档日志

- 2026-07-18 归档
