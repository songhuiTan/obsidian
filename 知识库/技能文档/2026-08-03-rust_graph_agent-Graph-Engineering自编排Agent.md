# rust_graph_agent（rgraph）：自编排 Graph Agent Runtime，让 Agent 变成"会自己改图纸的工程队"

> 来源：小张｜老码小张 2026-08-03
> 原文：[我用 Rust 撸了个自编排的 Agent，它拥有了"自我完善"的能力，群友：这难道是传说中的Graph Engineering ？](https://mp.weixin.qq.com/s/YECoz0HODWz3CqC6D4SCDw)
> GitHub：[coder-brzhang/rust_graph_agent](https://github.com/coder-brzhang/rust_graph_agent)（9117 行 Rust / 6 个直接依赖）
> 标签：AI, Graph Engineering, Agent, Rust, 运行时, 编排, 深度技术

---

## 概述

老码小张用 Rust 实现的 **Graph Agent Runtime（rgraph）**——一个由确定性 Rust 内核驱动、允许 LLM 动态修图、支持多 Agent Loop 并行协作、可中断恢复的执行引擎。

**一句话定位：模型决定"建议做什么"，Rust 决定"是否允许、何时执行、执行到什么程度"。**

这不是聊天机器人，不是单 Loop Agent，也不是 LangGraph 那种"用代码画图"的编排框架。前两篇长文（harness 工程、loop 工程）都默认"一个 Agent 能搞定一个任务"，这篇回答的是：怎么把一个 Agent 变成一支工程队。

## Agent 工程的五个阶段

| 阶段 | 核心问题 | 关键能力 |
|:--|:--|:--|
| **Prompt Engineering** | 怎么让 AI 听懂人话 | 提示词、角色、约束、示例、输出格式 |
| **Context Engineering** | 怎么让 AI 拿到对的资料 | 上下文选择、记忆、检索、压缩 |
| **Harness Engineering** | 怎么让 AI 在真实系统里稳定跑 | 工具协议、超时、重试、沙箱、日志 |
| **Loop Engineering** | 怎么让 AI 自己连续工作 | 规划→执行→观察→反思→修正，停止条件，预算 |
| **Graph Engineering** | 怎么编排多个 Loop/Agent 完成复杂任务 | 任务拆解、依赖、并行、分支、汇聚、**动态修图**、失败恢复 |

前四个阶段武装"一个 Agent"，第五个阶段是**质变**：模型从干"执行"变成干"协作"。一个 Agent 再聪明也只是高级打工人，一支会协作的工程队才是一个组织。

**核心判断：Agent 工程的下一站，不是更聪明的模型，是把多个 Agent 编排成一支能自己改图纸的工程队。**

## 业界三大流派（横向对比）

| 流派 | 代表 | 优点 | 缺点 |
|------|------|------|------|
| **代码定义图** | LangGraph、Pydantic Graph | 灵活 | 图是静态的，跑起来就固定了 |
| **消息驱动协奏** | AutoGen、CrewAI、MetaGPT | 涌现感强 | 不可控：不知道何时停、谁在干嘛，调试噩梦 |
| **人画图+单 Loop 执行** | n8n、Dify、Coze | 门槛低 | 节点不是 Agent，没有 Loop 能力 |

三者都缺一样东西：**图本身能在执行中被 Agent 改。**

rgraph 选第四条路：**LLM 负责规划、提议、修正；Rust 内核负责编译、校验、调度、合并、恢复。模型是建议者，Rust 是裁判。** 两者解耦，是 Graph Engineering 能稳定上生产的根本。

## 五个核心设计

### 1️⃣ 让 LLM 无直接控制权（灵魂设计）

LLM 只能返回五种结构化对象：`GraphProposal / GraphPatch / AgentAction / NodeResult / StatePatch`，Rust 内核统一校验后再执行。

```rust
pub enum AgentAction {
    CallTool { tool_id: String, arguments: Value },
    Complete { result: Value, state_patch: StatePatch, graph_patch: Option<GraphPatch> },
    Fail { reason: String, retryable: bool },
}
```

不接受自由文本控制命令。LLM 想"顺手删个文件"——做不到，没有权限。只能通过 `CallTool` 调授权工具，工具权限是**节点 capabilities ∩ Agent 配置 ∩ 全局配置三层交集**。**模型负责想，Rust 负责兜底。**

### 2️⃣ 动态修图（GraphPatch）—— Agent 在跑的过程中能改图纸

这是最硬核的功能，与 LangGraph 静态图的本质区别。

场景：架构评审时 critic 发现"内存有风险但没分析"，传统做法是踢回给人或重启流程。rgraph 的做法：critic 在 Complete 时附上 `graph_patch`——`add_node(memory_risk)` + `add_edge(final_report → memory_risk)`。

模型只是提议，Rust 内核严格校验：
- `base_revision` 必须与当前版本一致（防基于陈旧图改）
- 不能修改已 Succeeded/Failed 的节点（C-15）
- 不能创建非法环（Kahn 算法检测）
- 不能引入未注册的 Agent 或 Tool
- 单次新增节点 ≤16，patch 操作 ≤32，图版本 ≤16

校验通过 → 复制候选图 → **重新编译**（拓扑/环/写冲突/可达性）→ bump revision → 写 `graph.v0002.md` → 切换版本 → 新节点跑起来。校验不过 → 写 `GraphPatchRejected` 事件，但**节点不因此失败**，critic 可重新规划。

**Agent 拥有了"自我完善"的能力，但这种能力是被约束的**——不能乱改历史、不能改畸形、不能烧穿预算。

### 3️⃣ 并行确定性（Graph Engineering 最难的工程问题）

传统并发用加锁，但 Agent 场景加锁会失去并行意义。rgraph 把"读"和"写"彻底分开：

> 节点不能直接持有全局状态的可变引用。只能：1. 读取不可变 StateSnapshot；2. 返回 StatePatch；3. 由确定性 Reducer 应用补丁。

每个节点声明 `reads` / `writes`，写路径不冲突就编译时允许并行（C-12）。合并顺序固定，与完成顺序无关：

```
graph_revision → topo_rank → priority → node_id → attempt
```

Reducer 类型：`Replace / AppendArray / MergeObject / SetUnion / MaxNumber / MinNumber`。

**硬保证（SPEC G-01）：同一份图、相同输入、相同节点输出事件，得到相同的状态演进与调度结果。** 跑多少次都一样 → 可以 **replay**，出了 bug 能复现。

### 4️⃣ 事件存储 + 恢复（不要数据库）

**JSONL 事件存储：** 所有状态变化先写 `session.jsonl` 再更新内存。追加写入、不修改、不删除。关键事件写完后 `sync_data` 强制落盘。**没有数据库**——Agent 一次 run 的事件只有几十 KB 到几 MB，序列化 JSONL 就够；数据库的连接池/迁移/并发是过度设计。

**resume：** 进程退出后 `rgraph resume <run_id>`：
1. 读 session.jsonl → 找最后一个完整 Checkpoint → 从该事件后重放
2. 重建 RunState（图版本、节点状态、全局状态、预算）
3. Running 节点标记 Interrupted 重新 Pending
4. **Succeeded 节点永不重跑**——跑了 200 步、8 万 token 的成果不能丢

作者实测：DeepSeek API 上 kill -9 进程，resume 后接着跑，最终结果与不 kill 一致。

**哲学：在 Agent 场景里，数据库是负债，追加写入的日志才是基础设施。** 手写 60 行 Kahn 拓扑排序，比引 petgraph 追版本更值——依赖越少越值钱。

### 5️⃣ 六重约束的 Agent Loop + 两层预算

```rust
pub struct LoopPolicy {
    pub max_steps: u32,        // 步数上限
    pub max_model_calls: u32,  // 模型调用次数
    pub max_tool_calls: u32,   // 工具调用次数
    pub max_tokens: u64,       // token 预算
    pub timeout_ms: u64,       // 超时
    pub no_progress_limit: u32, // 无进展上限（最值钱）
}
```

`no_progress_limit` 防"自言自语"——LLM 在"该不该再做点什么"和"算了"之间反复横跳烧 token。五条无进展信号：连续同一工具同参数、连续两次观察摘要一致、StatePatch 为空、无新增 Artifact、重复相同错误。

预算两层：**全局预算 + 节点预算**。节点预算不得超过全局剩余；多个 Ready 节点预算总和超剩余时按优先级排队。**预算不是省钱，是给 Agent 设的硬护栏。**

## 单 Loop Agent vs Graph Agent

| 维度 | 单 Loop Agent | Graph Agent（rgraph） |
|:--|:--|:--|
| 任务复杂度 | 一个目标，一条路径 | 一个目标，多分支并行+Join |
| 控制流 | LLM 决定每一步 | 图决定大结构，LLM 只决定节点内 |
| 状态管理 | 一个会话窗口 | 每节点独立窗口+reads/writes 声明 |
| 失败恢复 | 整个 run 重来 | 单节点重试，Succeeded 不重跑 |
| 预算控制 | 一个池子 | 全局预算+节点预算两层 |
| 可观测性 | 看消息流 | 看图节点状态+JSONL 事件流 |
| 改图能力 | 不存在 | 运行中可由 Critic 提交 GraphPatch |
| 上下文成本 | 累积爆炸 | 节点隔离，只读 reads 路径 |
| 调试难度 | 黑盒 | replay 重放，确定性复现 |

**决策标准：** 翻译/摘要/单轮问答/单文件改写——单 Loop 够用，别上 graph（过度工程）。评审技术方案/调研领域出报告/重构模块——天然有结构需要拆，单 Loop 不够。

## 什么时候别这么做

1. **不是所有任务都需要 Graph Engineering**——工程复杂度应匹配业务复杂度，周末 demo 别上 rgraph
2. **动态修图对模型要求高**——弱模型提不出合法 GraphPatch（忘填 base_revision、试图改已完成节点、非法 JSON）。GraphPatch 不是免费午餐，它把规划负担转嫁给了模型能力

## 战略分析

### 与现有体系的对照

| 维度 | rgraph | LangGraph | Hermes Agent | LoopX（已归档） |
|------|--------|-----------|-------------|----------------|
| 图 | **运行中可动态修图** | 静态代码定义 | 无图概念 | 状态内核非图 |
| 执行 | Rust 确定性内核 | Python | Agent 核心 | 不做执行 |
| 控制权 | LLM 只能提建议 | LLM 参与 | Skill 约束 | 状态约束 |
| 状态 | StatePatch+确定性 Reducer | StateGraph | 上下文窗口 | 六块状态外置 |
| 持久化 | JSONL 事件存储 | Checkpointer | 会话记录 | — |
| 恢复 | resume 不重跑 Succeeded | 断点恢复 | 部分 | Handoff/Recovery |
| 并行 | reads/writes 声明式确定性并行 | fan-out/fan-in | delegate 并行 | — |

### 高价值启示

1. **"LLM 无直接控制权"是 Graph Engineering 的灵魂**。模型是建议者、内核是裁判，两者解耦才能上生产。这和 LoopX 的"状态外置"、Harness 的"硬驱动"是同一条主线：**控制权从模型手里回到工程手里**。

2. **动态修图是真正往前走的一步**。作者明说"这个能力是我之前在 Hermes 上一直没做出来的。LangGraph 也没做。AutoGen 也没做。"——GraphPatch + 严格校验（base_revision/环检测/版本上限）是 rgraph 的差异化价值。

3. **JSONL 事件存储反数据库**——和 LoopX 的"结构化数据是稳定依靠"呼应：Agent 场景数据库是负债，追加日志才是基础设施。Hermes 会话记录已用 JSONL，方向一致。

4. **确定性 = 可 replay = 可复现**——SPEC G-01 硬保证让生产排障成为可能。这是单 Loop Agent 无法给的能力。

5. **no_progress_limit 防"自言自语"**——比 max_steps 高级：前者防暴走，后者防烧 token。生产 Agent 翻车一半是自言自语。

### 能力差距与判断

- Hermes 缺 graph 层和动态修图能力，但作者正是 Hermes 用户（"之前在 Hermes 上没做出来"），说明这是社区对 Hermes 的期待方向之一
- 动态修图对弱模型不友好——用 DeepSeek-v4-flash 跑得通，更小模型需自测
- 项目仅在小群分享（600 人），非完全公开传播，GitHub 地址已给出

## 关联阅读

- [[2026-07-18-Lisp-Agent-OS-比Rust-Hermes更强大-计划可见技能可编程经验可沉淀运行可回放]] — 同作者（老码小张）的 Agent OS 构想
- [[2026-07-25-Harness引擎-自研编排工具-云淡风轻]] — Harness 层设计与本文第一至四阶段对应
- [[2026-08-02-LoopX-超长程Agent管理工具-状态内核]] — 长程状态管理，与本文第五阶段互补
- [[2026-07-12-LangGraph讲透系列06-并行分支fan-out与join]] — LangGraph 静态图并行 vs rgraph 动态修图

## 归档日志

- 2026-08-03 归档
