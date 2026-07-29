# 用阿里 AgentScope 复刻了一个 WorkBuddy

> 来源：叶小钗｜公众号 2026-07-20
> 原文：[我用阿里 AgentScope 复刻了一个 WorkBuddy](https://mp.weixin.qq.com/s/MjH1zVaEAkmGCrSyvP1h7g)
> GitHub：[AgentScope-AI/agentscope](https://github.com/agentscope-ai/agentscope)
> 标签：AI, Agent, WorkBuddy, AgentScope, 框架, 工具管理, 权限, 生产级

---

## 概述

叶小钗利用阿里开源框架 **AgentScope** 实现了一个 mini-WorkBuddy。文章不是框架的 API 文档式介绍，而是以"用真实需求驱动框架学习"的方式，逐步拆解 Agent 开发的完整链路：模型配置、工具管理、工作目录、权限系统、工具审批流和流式事件消费。每个环节都同时覆盖 AgentScope 的框架设计和 mini-WorkBuddy 的产品化决策。

## AgentScope 生态概览

| 项目 | 定位 |
|------|------|
| **AgentScope** (Python) | Agent 开发框架，封装模型调用、消息格式、工具执行、记忆、多 Agent 通信 |
| **AgentScope Runtime** | Agent 运行时——安全稳定地对外提供服务 |
| **AgentScope Java** | Java/JVM 技术栈的 Agent 框架，适合 Spring Boot 项目 |
| **AgentScope Studio** | 调试与追踪工具 |
| **AgentScope Samples** | 示例应用集合 |

核心解决：如何把大模型、提示词、工具、记忆和执行流程组合起来，形成一个能持续推理并执行任务的完整 Agent。

## 模型配置

AgentScope 模型层由 **Credential**（API 认证凭证）和 **ChatModel**（模型族）组成。但在产品化场景中需要额外一层模型配置管理：

```
用户页面配置 → workspace/models.json 保存 → 后端 create_model_client() 适配 → AgentScope Credential + ChatModel
```

关键设计：

| 问题 | 解决方案 |
|------|----------|
| 用户切换模型 | 模型配置 ID 放进 Agent 缓存 key，切换后缓存不命中，自动创建新模型 Agent |
| api_key 安全 | 前端读取模型列表时后端主动移除 api_key |
| 多厂商适配 | 统一字段（provider/model/api_key/base_url/thinking/max_context），按 provider 选择 AgentScope 对应实现 |

```
create_model_client() 适配模式：DashScope → DashScopeChatWrapper，DeepSeek → 对应 ChatModel，以此类推
```

## 工具管理与 Toolkit

### Toolkit 的四类内容

| 类别 | 说明 |
|------|------|
| 普通 Tool | Read、Write、Bash 等内置工具 |
| MCP Client | 远程工具 |
| Agent Skills + Skill Loader | 技能加载 |
| Tool Group | 按组启用/停用工具 |

`Toolkit(tools=[...], skills_or_loaders=[...], mcps=[...], tool_groups=[...])`

- `basic` 基础工具组始终可用（包括 tools、skills、mcps）
- 额外 `tool_groups` 可动态启用/停用，避免一次把所有工具 Schema 塞给模型

### 工具执行流程

```
模型返回 ToolCall → Toolkit.call_tool()
  1. 检查工具是否存在及工具组是否激活
  2. 解析 JSON 参数
  3. 需要时注入 AgentState
  4. 调用同步/异步函数或生成器
  5. 增量结果统一转 ToolChunk
  6. 汇总为完整 ToolResponse
```

### mini-WorkBuddy 的工具选型

显式选择了六个通用工具：Read、Write、Edit、Glob、Grep、Bash（Kill 和 Run 等任务相关工具留给专家团管理）。创建 Agent 时传入 Toolkit：

```python
agent = Agent(
    name="WorkBuddy",
    system_prompt=SYSTEM_PROMPT,
    model=model,
    toolkit=toolkit,
)
```

## 工作目录与权限

### 工作目录三层作用

| 层面 | 实现 | 作用 |
|------|------|------|
| 默认位置 | `Bash(cwd=workdir)` | Bash 启动时的当前目录 |
| 权限范围 | `PermissionContext.working_directories` | Write、Edit、文件类 Bash 的权限判断 |
| 会话隔离 | workspace_id ↔ workspace_dir | 不同 WorkBuddy 工作空间的聊天数据隔离 |

额外登记目录：长期记忆目录和 Skill 目录（Agent 执行任务时可能需要读取）。

### AgentScope 的 PermissionMode 五种模式

| 模式 | 行为 | 适用场景 |
|------|------|----------|
| DEFAULT | 严格交互，未明确允许则 ASK | 需要用户全程确认 |
| ACCEPT_EDITS | 工作目录内写入自动放行，目录外仍 ASK 或 DENY | 日常开发使用 |
| EXPLORE | 严格只读，Write/Edit 直接拒绝 | 阅读仓库不给修改权限 |
| BYPASS | 跳过所有确认（含安全风险） | 沙箱/容器/完全信任环境 |
| DONT_ASK | 需要询问的转拒绝，不弹框 | 后台定时任务/无人值守 |

### mini-WorkBuddy 简化为三档

| 用户可见模式 | 映射的 AgentScope 模式 | 行为 |
|-------------|----------------------|------|
| 默认 | DEFAULT | 严格交互 |
| 自动审批 | ACCEPT_EDITS | 工作目录内自动放行 |
| 完全放行 | BYPASS | 跳过确认（高风险，几乎不用） |

实际使用中"自动审批"是主力档——默认模式需要频繁点击确认，体验差。

### PermissionRule 自定义规则

```python
context = agent.state.permission_context
engine = PermissionEngine(context)
engine.add_rule(PermissionRule(
    tool_name="Bash",
    rule_content="uv run pytest",
    behavior=ALLOW,
    source="userSettings"
))
```

规则字段：`tool_name`、`rule_content`（命令/路径，None 表示匹配全部）、`behavior`（ALLOW/DENY/ASK）、`source`。

## 工具审批流程

```
Agent 执行 → 权限引擎返回 ASK
  → AgentState 标记为 ASKING
  → 产生 RequireUserConfirmEvent
  → 后端暂停执行，向页面发送 approval_required
  → 用户确认后返回 UserConfirmResultEvent
  → 原 Agent 继续 ReAct 流程
```

### PendingApproval 设计

mini-WorkBuddy 自定义的数据结构，不是 AgentScope 提供的类型：

- 记录定位原 Agent 的信息、待确认的工具调用、原请求配置、已产生的流式内容
- 目前只保存在后端进程内存中（服务重启后无法继续）
- 支持并行工具调用批量确认（ConfirmationBatch）
- 拒绝结果回到 Agent，模型可改用安全方法或说明权限不足

## 工具执行过程流式显示

AgentScope 通过事件机制流式输出工具执行状态：

| 事件 | 内容 |
|------|------|
| ToolCallStartEvent | 工具名称与调用参数 |
| ToolCallDeltaEvent | 参数片段 |
| ToolCallEndEvent | 参数完整后发送 |
| ToolResult 聚合 | 结果聚合后发送 tool_result |
| RequireUserConfirmEvent | 待审批的工具调用完整信息 |

前端按 tool call ID 创建/更新工具卡片，显示工具名称、格式化参数、执行状态和结果。

### 工具分层架构（mini-WorkBuddy）

| 层 | 内容 |
|----|------|
| 基础工具 | Read/Write/Edit/Glob/Grep/Bash |
| 技能系统 | Skills 和 Skill Loader |
| MCP 工具 | MCP Client 提供的远程工具 |
| 专家团工具 | 任务相关工具，通过 Tool Group 按组管理 |

## 关键洞察

1. **AgentScope 的 Toolkit 不是工具数组**——它是工具发现、分组、MCP 接入、Skill 加载和统一执行的管理器。空 Toolkit 不自带任何工具，必须显式注册。
2. **工作目录的 cwd ≠ 安全边界**。Bash 的 cwd 只决定命令起始位置，不做安全限制。真正的权限判断在 `PermissionContext.working_directories` 中。
3. **PermissionMode 映射到用户侧需要简化**。AgentScope 的 5 种模式粒度太细，mini-WorkBuddy 简化为 3 档（默认/自动审批/完全放行），实际常用只有"自动审批"。
4. **工具审批恢复需要应用层自行实现**。AgentScope 提供 RequireUserConfirmEvent 暂停执行，但恢复所需的 PendingApproval 数据结构不在框架中，需要在应用层自行管理。
5. **拒绝也是 Agent 的正常反馈**。用户拒绝工具执行后，Agent 可以改用替代方案或说明权限不足，而不是卡死在弹窗上。

## 关联阅读

- [[2026-07-15-拆完WorkBuddy-我看到了生产级Agent的完整形态-叶小钗]] — 对 WorkBuddy 本身的产品和工程架构拆解（本文是 AgentScope 视角的实践复刻）
- [[2026-07-12-生产级Agent全景-架构Harness工程组织与人才-叶小钗]] — 生产级 Agent 全景方法论

## 战略分析

### 与现有框架的对照

本文揭示的 AgentScope 设计哲学（Toolkit 即容器+执行器、Permission 分层引擎、事件流式工具状态）与当前体系有多个交叉点：

| 维度 | AgentScope | Hermes Agent | 差异点 |
|------|-----------|-------------|--------|
| 工具管理 | Toolkit（容器+执行+分组+SKill+ MCP） | config.yaml 注册 + 工具发现 | AgentScope 的 Toolkit 更像中心化"工具 OS" |
| 权限系统 | 5 种 PermissionMode + PermissionRule 引擎 | 无内置权限系统 | 值得借鉴：特别是不问/只读两种模式对有安全需求的任务有价值 |
| 审批流程 | RequireUserConfirmEvent 暂停 → 恢复 | 无 HITL 审批 | 如果未来 Hermes 需要 HITL，这个事件驱动的审批模式是参考方案 |
| 模型适配 | create_model_client 适配器模式 | config.yaml 多 provider | 思路一致，实现方式不同 |
| Skill 加载 | Skill Loader 整合进 Toolkit | SKILL.md 文件体系 | AgentScope 的 Skill 是 Toolkit 的子集，更受控 |

### 对当前体系的启示

1. **权限分层设计值得引入**。当前 Hermes 没有内置的"只读模式"或"工作目录隔离"。AgentScope 的 EXPLORE（只读）和 ACCEPT_EDITS（工作目录内放行）两种模式直接对应到"先审查再执行"和"日常自动审批"两种工作流。如果 Hermes 的 terminal/file 工具要支持类似的安全层级，PermissionMode + PermissionRule 的双层设计是成熟参考。

2. **Toolkit 作为工具 OS 的抽象层**。AgentScope 把 Tool、MCP、Skill 统一在 Toolkit 下的设计，比分散管理更一致。当前 Hermes 的 Skill 体系（SKILL.md）和工具注册（config.yaml tools）是两条线，AgentScope 的 `tool_groups` 动态启用/停用机制也是一个有用的"工具按场景折叠"模式。

3. **工具审批流的产品化启示**。PendingApproval + ConfirmationBatch 的设计虽然是 mini-WorkBuddy 自建的，但其"暂停 Agent → 用户确认 → 恢复 Agent"的闭环和拒绝结果回路的处理方式，是生产级 HITL 的参考实现。

## 归档日志

- 2026-07-24 归档
