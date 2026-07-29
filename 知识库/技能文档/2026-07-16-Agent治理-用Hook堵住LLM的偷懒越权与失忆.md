---
title: "Agent 治理：用 Hook 堵住 LLM 的偷懒、越权与失忆"
source: "腾讯技术工程"
source_url: "https://mp.weixin.qq.com/s/ISwjIw5lj7JlcQJV7BOx5g"
date: "2026-07-16"
tags: [Agent, 治理, Hook, HITL, 护栏, DECO, 腾讯]
---

> prompt 管不住的，框架来堵。本文是 DECO（腾讯数仓 Agent 引擎）实践系列之一，聚焦护栏层：用 Hook 切面在代码层确定性兜底三类 LLM 生产问题。

## 三类生产问题

| 问题 | 表现 | 根因 |
|------|------|------|
| **偷懒** | 长 SQL 截断、占位略写（`-- 其他字段...`）、复印重写到 token 耗尽 | 物理 token 预算不足 |
| **越权** | 用户没确认就调了发布工具，把讨论中的表结构推上生产 | 模型无法区分"查询"和"发布"的可逆性差异 |
| **失忆** | 改了表不分析下游风险、产出了图不知告诉用户 | 模型追求最短路径，不会主动加检查步骤 |

> 在 prompt 里多写几句 ⚠️ 禁止根本管不住。唯一的解法是在 Agent 框架层，让偷懒和越权的路径代码级强制走不通，让失忆的已知盲区确定性补齐。

---

## 一、Hook 链：在关键切面挂载护栏逻辑

Hook（Callback）切面是护栏的基础设施。DECO 主要用到四个切面：

| 切面 | 触发时机 | 用途 |
|------|---------|------|
| `beforeTool` | 工具真正执行前 | 长脚本回写前从文件加载全文、HITL 门禁 |
| `afterTool` | 工具执行后、结果回 LLM 前 | 长脚本拉取后替换为引用句柄 |
| `before/afterModel` | 每次请求 LLM 前/后 | 响应用户取消等 |
| `before/afterAgent` | 单个 Agent 运行前/后 | 对话持久化等 |

**设计原则：基础设施和推理逻辑解耦**——Hook 切面上的逻辑独立运作，模型的 ReAct 循环不用感知；新增/删除一个 Hook，主流程不改一行代码。

## 二、长文本完整性护栏（治偷懒）

### 问题定位

长产物的偷懒是结构性问题。解法：**把 LLM 必须接触的长内容降到最少、每次接触窗口压到最小、所有写入路径都做成小步增量改 + 强制校验。**

### 方案：读写两侧 offload + 引用句柄

LLM 永远不直接接触脚本全文。两端都用 Hook 拦截 + 沙箱文件做中转站：

**拉取侧：Offload Hook（`afterTool`）**
- Hook 拦截含 `scriptContent` 的响应，全文写入沙箱只读快照
- 响应中替换为引用句柄：`<offloaded to /sandbox/{taskName}.remote.etl (read-only snapshot, length=N chars). To start editing, run copy_file(...) first, then str_replace.>`
- 失败降级：落盘失败则透传原内容，不阻塞主流程

**写回侧：Onload Hook（`beforeTool`）**
- 工具入参走 `scriptFilePath` 而非 `scriptContent`
- Hook 从沙箱加载全文覆盖入参，转发前剥离 `scriptFilePath` 字段
- 文件不存在/内容为空 → 抛异常阻断工具调用
- 工具协议：`scriptContent` 与 `scriptFilePath` 互补参数，下游无感知

### 效果

| 维度 | 治理前 | 治理后 |
|------|--------|--------|
| 修改任务工具调用输出 token | 每轮重传整段 SQL | **直降约 90%** |
| SQL 复印自截断 | "view→重写"路径下概率近 100% | 物理消除（只走 str_replace） |
| 写侧失败 | — | 阻断调用，杜绝发布残缺脚本 |

### 行业对比

| 工程 | 读侧 | 写侧 | 自动化程度 |
|------|------|------|-----------|
| ADK ArtifactService | save_artifact/load_artifact API | ❌ 需手动调 API | 手动 |
| LangGraph DeepAgents | 内置 Large Tool Result Offloading | ❌ 只做读侧 | ✅ 全自动 |
| DECO | 两端对称 offload | ✅ onload + 文件协议 | ✅ Hook 层自动 |

## 三、危险操作确认 HITL（封越权）

### 方案：配置驱动的 `beforeTool` 守卫

HITL 本质是一个特殊的 `beforeTool` Hook。危险工具清单是**配置出来的**，每个配一个授权标记和确认对话框：

```yaml
deco:
  dangerous-tools:
    - name: packCommit
      required-state: confirm_pack
      confirmation:
        title: "请确认发布方式"
        options:
          - {id: direct, label: "直接发布（免审批）", value: direct}
          - {id: approval, label: "提交审批", value: approval}
          - {id: draft, label: "保存草稿", value: draft}
```

守门流程：Agent 每次调工具 → 框架过守卫 → 判断是否危险操作 → 用户未授权则阻断 → 发送确认框 → 用户选择 → 写入 session.state → 续跑请求 → 守卫放行。

> 必须框架层拦，不能信 LLM。确认动作只能由真实用户在前端触发。

### 自研 HITL 的必要性

| 维模 | ADK ToolConfirmation | DECO DangerousToolGuard |
|------|---------------------|------------------------|
| 触发 | 工具内部主动暂停 | beforeTool 切面外部拦截 |
| 危险清单 | 工具代码里声明 | 配置驱动（yaml） |
| 交互 | yes/no | 多选项 + 带输入控件 + 变更预览 |
| 业务集成 | 工具级通用拦截 | 业务级集成确认（发布前展示变更清单） |

自研必要性不在于"框架没有 HITL"，而在于"框架的 HITL 不够业务化"。

## 四、上下文联动闭环（补失忆）

### 范式：Hook 管采集，Attachment 管注入

```
工具调用 → Hook(afterTool) 采集事实 → 写入 state → Attachment 注入下一轮 prompt
```

| 方案 | 可靠性 | LLM 偷懒风险 |
|------|--------|-------------|
| prompt 里写"改表后记得分析风险" | ❌ 软约束 | ✅ 高 |
| 单独发一轮"请分析风险" | 🟡 依赖调度 | ✅ 中 |
| **Hook→state→Attachment** | ✅ 确定触发 | ❌ 零 |

### 案例一：RiskAnalysisHook
- 挂载点：`afterTool`
- 触发条件：`upsertTable` 带 `tableId` 参数（新建表不带 `tableId` 直接跳过）
- 行为：查询下游依赖表 → 风险分析结果写入 state → 下一轮 Attachment 注入 prompt
- 关键设计：判定条件精确（带 tableId 才是改表）、累积写入（多次改表一次性汇总）

### 案例二：PythonImageHook
- 挂载点：`beforeTool` + `afterTool`
- 行为：beforeTool 加文件快照 → afterTool 对比 → 发现新 .png/.jpg/.svg → 生成预签名 URL → 写入 state → Attachment 注入
- 关键设计：前后文件快照对比比 LLM 用 `bash ls` 可靠得多

## 五、Hook 全景生态

DECO 在四个切面上挂了十余个 Hook，全部遵循同一条原则：**不改业务循环、不动工具实现，把横切逻辑挂在切面上。**

主要分类：

| 分类 | Hook | 挂载点 |
|------|------|--------|
| 长文本护栏 | TaskScriptOffload/Onload、TableColumnsOffload、DdlBodyOffload | afterTool/beforeTool |
| 危险操作 | DangerousToolGuard | beforeTool |
| 工具返回处理 | LineageResponseOffload、ToolResponseTruncator、ToolResponseFormatter | afterTool |
| 可观测 | ToolCallLogHook、LoggingHook、ConversationPersistenceHook | 多点 |
| 前端刷新 | SqlExecuteHook、CopyFileHook、ReleaseItemCollectorHook | before/afterTool |
| 上下文联动 | RiskAnalysisHook、PythonImageHook | before/afterTool |
| 沙箱环境 | EnvVarCaptureHook | afterTool |

### 结语的一句话

> **prompt 定意图，Skill 定规矩，框架 Hook 定边界——能用确定性兜底的，别交给模型。**

---

## 战略分析

### 与当前体系的深度对照

这篇文章是近期归档中**与 Hermes 体系最直接相关的工程实践文章**。DECO 作为腾讯数仓 Agent 引擎面临的"偷懒、越权、失忆"三类问题，正是所有生产级 Agent 的通用痛点，我们的体系也不例外。

**1. Hook 链设计 vs Hermes Skill 体系**

DECO 的 `beforeTool`/`afterTool` 四个切面与 Hermes 的 cronjob 执行链在抽象层面是同一类思想——在 Agent 执行的关键节点插入横切逻辑。不同的是 DECO 的 Hook 挂载在框架内部（ADK），而我们的横切逻辑分布在 skill 定义和 cron 调度中，缺少统一的切面管理。

**2. 长文本 offload → 直接映射到我们的场景**

Hermes Agent 在处理长文档归档、长上下文对话时，同样面临 token 预算和 LLM 截断的问题。DECO 的"引用句柄 + str_replace 小步改"模式非常值得借鉴——特别是在 kb-archive-article 流程中处理超长文章时，可以先落盘、再只传引用句柄。

**3. HITL 确认 → 微信交互的天然优势**

我们的微信端交互天然具备"框架层 HITL"的条件——每次工具调用都可以在微信端弹确认。目前部分高风险操作（如文件写入、技能删除）已有基础确认，但没有系统化为 DECO 这种配置驱动的危险工具守卫。

**4. Hook→state→Attachment 闭环 → Memory 系统的启示**

DECO 的"事件→采集→注入"主动流水线，比"存了再读"的被动模式更适合生产场景。这提示我们 TencentDB Memory 的 L0-L3 四层记忆系统可以增加一个"主动事件注入"层：在某些关键工具调用后，自动将分析结果注入下一轮上下文，而不是等着 LLM 主动查询 memory。

**5. 最值得抄的架构决策**

> "prompt 管不住的，框架来堵"——这是整篇最值钱的一句话。

对照我们的体系，以下能力目前还依赖 prompt 软约束，应该逐步转移到框架层：
- 长文本截断保护 → offload 机制
- 危险操作确认 → HITL 守卫
- 遗漏步骤补全 → afterTool 主动采集 + 上下文注入

### 与之前归档文章的关系

- **叶小钗 WorkBuddy 篇**讲宏观架构（六阶段、双视角）
- **Jiong 五层架构篇**讲业务编排层（Capability Router）
- **腾讯 DECO 这篇**讲护栏层落地（Hook 挂载点的具体实现）

三篇从"架构是什么"到"业务怎么编排"到"异常怎么兜底"，构成了完整的生产级 Agent 工程链路。

## 归档日志

- 2026-07-16 归档（腾讯技术工程，作者 xiangnzhang，DECO 实践系列之一，原文发布日期未标明）
