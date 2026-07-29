---
title: "Function Call、MCP 与工具治理：从"模型会调用工具"到企业级 Agent 工具体系"
author: "银河技术（Ray的银河技术）"
source_url: "https://mp.weixin.qq.com/s/1v5Ljz_ov6NIOcttRFKkCw"
date: "2026-06-28"
series: "新一代 Agent 系统架构与工程实践（第7篇）"
tags: [Function Call, MCP, Tool Governance, Agent 架构, 工具治理, 企业级, Spring Boot, Tool Registry]
---

# Function Call、MCP 与工具治理：从"模型会调用工具"到企业级 Agent 工具体系

> **专题：新一代 Agent 系统架构与工程实践｜第 7 篇**
>
> 前六篇分别讨论了 Agent 架构三底座（Skills、Harness、Loop），及 RAG 工具化和 ReAct 与 Loop 的关系。本文聚焦 Agent 与外部世界交互的关键层：**模型如何发现工具、表达调用意图、连接外部系统，并在企业环境中受统一治理。**
>
> **核心结论：**
> - **Function Call 是模型与应用之间的工具调用机制**
> - **MCP 是 AI 应用与外部工具的标准化连接协议**
> - **工具治理是覆盖注册、发现、权限、安全、版本、执行、审计和下线的完整工程体系**

---

## 一、Function Call 是什么

**Function Call = 模型结构化表达 "我想调用这个工具，用这些参数"**

- 模型不直接执行函数，只返回调用意图（工具名 + 参数 JSON）
- 应用运行时负责：解析 → 校验 → 权限检查 → 执行 → 结果返回模型
- **模型提出调用请求，应用决定是否真正执行**

```
用户请求 + 工具定义 → 模型 → Function Call {name, arguments}
→ 应用：校验/权限/凭证 → 执行真实业务 → 结果返回模型 → 最终回答
```

**Function Call 不解决：** 工具共享、协议标准化、权限、审计、版本、审批、供应链安全等问题。

---

## 二、MCP 是什么

**MCP（Model Context Protocol）= AI 应用与外部系统的开放连接协议**

### MCP Server 暴露三类核心能力
| 能力 | 说明 | 示例 |
|------|------|------|
| **Tools** | 可被调用的执行能力 | `ticket.create`, `database.query` |
| **Resources** | 可读取的上下文数据 | `file:///README.md`, `git://.../OrderService.java` |
| **Prompts** | 参数化模板或工作流入口 | `review_pull_request`, `analyze_incident` |

### MCP 解决了什么
将 "N 个 AI Host × M 个系统" 的复杂连接简化为：
- AI Host 实现 MCP Client
- 外部系统实现 MCP Server

### MCP 不是什么
MCP 不是：大模型、Agent Loop、RAG 算法、工作流引擎、权限中心、API Gateway、安全沙箱。
**MCP 标准化"如何通信"，但不替企业完成安全治理。**

---

## 三、Function Call 与 MCP 的关系

| 维度 | Function Call | MCP |
|------|---------------|-----|
| 核心目标 | 模型结构化表达工具调用 | 标准化 AI 应用与外部服务连接 |
| 主要边界 | 模型 ↔ Agent Runtime | MCP Client ↔ MCP Server |
| 工具发现 | 由应用提供定义 | 可通过协议列出服务端能力 |
| 是否自动安全 | 否 | 否 |
| 是否替代治理 | 否 | 否 |

**完整链路：**
```
模型 → Function Call → Tool Gateway → Schema 校验 → Policy & Approval
→ Credential Broker → Idempotency → MCP Client → MCP Server / 业务系统
→ 输出校验 → Trace & Audit → Observation → 模型
```

---

## 四、为什么需要统一工具治理

### 4.1 工具数量会快速膨胀
多个团队各自构建 Agent 时出现重复工具（`order.query` / `get_order` / `queryOrder`），参数/权限/版本各不相同。无注册治理 → 模型选择准确率下降、Token 增加、错误调用难追踪。

### 4.2 工具是 Agent 的真实权限边界
工具接入后 Agent 才能：读敏感数据、改生产配置、发消息、创建资源、操作资金。**工具治理 = Agent 权限模型的核心。**

### 4.3 工具错误产生真实副作用
一个错误工具调用可能：重复扣款、误删数据、发错邮件、改生产环境、泄露跨租户信息。

### 4.4 工具描述也是执行控制的一部分
模型依赖名称和描述选择工具。模糊描述（`name: process, description: 处理业务数据`）导致难以判断。工具定义需要像 API 契约一样被评审和版本化。

---

## 五、企业级工具治理总体架构

### 控制面与执行面分离

| 控制面 | 执行面 |
|--------|--------|
| 工具注册、元数据、所有者 | Schema 校验、真实调用 |
| 版本、生命周期、风险等级 | MCP 连接、凭证注入 |
| 权限策略、工具发现 | 超时、重试、限流、幂等 |
| 质量评测、依赖关系 | 沙箱、结果标准化、审计 |
| 下线影响分析 | Trace/Audit 写入 |

---

## 六、Tool Registry：工具注册中心

### 完整 Tool Definition 示例
```json
{
  "toolId": "order.cancel",
  "displayName": "取消订单",
  "protocol": "MCP",
  "riskLevel": "HIGH",
  "sideEffect": "WRITE",
  "idempotency": "REQUIRED",
  "approvalPolicy": "ORDER_CANCEL_APPROVAL",
  "requiredScopes": ["order:cancel"]
}
```

### 工具生命周期
```
DRAFT → REVIEWING → REJECTED/ACTIVE → DEPRECATED → DISABLED/SUSPENDED
```

### 连接信息独立管理
工具定义不保存密码。`credential_ref` 只引用密钥管理系统。

---

## 七、工具命名与语义设计

- **推荐格式：** `namespace.resource.action`（如 `order.cancel`, `ticket.comment.add`）
- **面向业务语义：** 不要暴露 `http.request` / `shell.execute` / `database.execute_sql`
- **单一职责：** 一个工具只做一件事（不推荐 `manage_order(action, orderId, payload)`）
- **描述包含副作用：** "该操作会改变订单状态并可能触发退款"

---

## 八、工具 Schema 设计

- **输入必须严格：** 使用 `pattern`、`enum`、`minimum`、`additionalProperties: false`
- **输出应有契约：** Agent 需要凭输出结构验证结果、判断后续步骤
- **参数区分来源：** 模型提供业务参数，系统注入 `tenantId`/`userId`/`traceId`/`idempotencyKey`

---

## 九、工具发现与按需加载

工具不要全部放入上下文——消耗 Token、降低准确率、增加攻击面。

**分层发现：**
```
L0: 仅提供工具类别和检索工具
L1: 根据任务识别候选领域
L2: 加载候选工具摘要
L3: 选中后加载完整 Schema
```

**Tool Search 元工具：** 允许任务动态搜索可用工具，结果经租户、Agent 白名单、用户权限、环境等多层过滤。

**Skills 声明工具权限：**
```yaml
allowed-tools:
  - metrics.timeseries.query
  - logs.search
forbidden-side-effects:
  - WRITE
```

---

## 十、MCP Server 接入治理

- **信任等级：** TRUSTED_INTERNAL → VERIFIED_VENDOR → RESTRICTED_EXTERNAL → UNTRUSTED_LOCAL
- **接入审核：** 所有者、协议版本、TLS、认证、工具列表、数据分类、安全扫描
- **工具快照与变更检测：** Server 的 `tools/list` 可能变化，变更需进入 `Detected → Diff → Review → Test → Publish`
- **本地 vs 远程：** 本地 MCP 适合 IDE/桌面；远程 MCP 适合企业共享服务（但需防 SSRF）

---

## 十一、Tool Gateway：统一执行入口

**Agent 不应直连 MCP Server。** 推荐：
```
Agent Runtime → Tool Gateway → MCP Client Pool → MCP Server
```

**16 步调用流程：** 解析 Tool ID → 校验 Schema → 加载身份 → Policy Engine → 审批 → 预算限流 → 幂等键 → 凭证代理 → 选择执行器 → 执行 → 输出校验 → 保存 Artifact → Trace & Audit → 返回 Observation

---

## 十二、权限与身份传递

三种身份不能混：
| 身份 | 说明 |
|------|------|
| 用户身份 | 谁发起任务 |
| Agent 身份 | 哪个 Agent 执行 |
| 服务身份 | Gateway 用什么凭证 |

**两种模式：** 用户委托（Token 代表用户）vs 服务账号（统一身份，权限易过大）

**Credential Broker：** 根据 `taskId + userId + agentId + toolId + resourceScope` 签发短期凭证。**密钥不进 Prompt、不进 Tool Arguments、不进模型上下文。**

---

## 十三、安全治理

- **Prompt Injection 防侧信道：** 工具输出可能含恶意指令 → 标记来源、不可信数据隔离、工具输出不修改系统策略
- **SSRF 防护：** 出站白名单、DNS 检查、IP 范围阻断、URL Scheme 限制
- **供应链安全：** MCP Server 可能更新后增加危险工具 → 固定版本、SBOM、容器隔离、工具快照、Kill Switch
- **最小权限：** 不要暴露 `shell.execute(root)`，用 `service.restart(allowed_service, environment)`

---

## 十四、副作用、幂等与恢复

| 场景 | 是否重试 | 处理方式 |
|------|---------|---------|
| 只读超时 | 可以 | 指数退避 |
| 参数错误 | 不可以原样重试 | 修正参数 |
| 写操作超时 | 不能直接重试 | 先回查状态 |

**Idempotency Key：** `taskId + stepId + logicalOperation + targetResource`

**UNKNOWN 状态：** 超时后 → 按幂等键回查 → 判定是否已执行

---

## 十五至十七、版本治理 & 错误模型 & Spring Boot 实现

**版本策略：** Tool ID + SemVer 分离，任务创建时固定版本，长任务恢复用原版本

**结构化错误：**
```json
{"code": "ORDER_ALREADY_SHIPPED", "category": "BUSINESS_CONSTRAINT",
 "retryable": false, "message": "订单已经发货，不能取消"}
```
**不向模型暴露：** 数据库连接串、内部 IP、密钥、完整堆栈

**Spring Boot 参考结构：** `tool-registry`, `tool-gateway`, `policy-engine`, `approval-service`, `credential-broker`, `mcp-client-manager`, `idempotency-service`, `audit-service`（附完整 Java 代码）

---

## 十八、Python Agent Worker 接入

Worker 不直接持有业务凭证，通过 Tool Gateway Client 统一调用：
```python
class ToolGatewayClient:
    async def invoke(self, task_id, step_id, tool_call, principal_token):
        # 向 Gateway 发起工具调用请求，由 Gateway 处理凭证/权限/幂等
```

---

## 十九至二十一、审批 & 可观测性 & 工具 Eval

**审批要点：** 审批的是 `Tool + Version + Arguments Hash + Target Resource`，参数变化后审批失效

**核心指标：** `tool_selection_accuracy`, `tool_latency_p95`, `tool_policy_denied_total`, `tool_cross_tenant_blocked_total`, `mcp_server_health`

**工具 Eval 四维度：** Tool Selection Eval（选对工具）、Argument Eval（填对参数）、Security Eval（抗注入）、Trajectory Eval（完整调用轨迹合理性）

---

## 二十二、常见错误实践

1. ❌ 认为接入 MCP 就自动实现安全
2. ❌ 将所有业务 API 原样暴露给模型
3. ❌ 一次加载所有工具
4. ❌ 用 Prompt 代替权限控制
5. ❌ 将密钥作为工具参数
6. ❌ 写操作超时后直接重试
7. ❌ MCP Server 工具变化自动生效
8. ❌ 工具描述随意填写
9. ❌ 只有调用日志，没有业务审计

---

## 二十三、选择建议

| 场景 | 方案 |
|------|------|
| 工具只在一个应用内部、数量少、延迟敏感 | 直接 Function Call |
| 工具需要被多个 AI Host 复用、跨语言、需标准化发现 | MCP |
| 高频核心工具 | 本地 Function Call / 内部 Tool Executor |
| 共享企业工具 | 内部 MCP Server |
| 外部 SaaS | 远程 MCP 或受控 Connector |
| 危险系统能力 | 专用 Gateway + Workflow + Approval |

---

## 二十四、演进路线

1. **本地 Function Call** — JSON Schema + 函数映射 + 基础 Tool Loop
2. **统一 Tool Definition** — 命名规范 + Registry + 风险等级 + 输入输出契约
3. **Tool Gateway** — 权限 + 审计 + 超时 + 限流 + 幂等 + 凭证代理
4. **MCP 接入** — Client Pool + Server + 连接管理 + 快照
5. **企业级治理** — 动态发现 + 版本依赖 + 审批 + 供应链安全 + Tool Eval + Kill Switch + 全链路 Trace

---

## 战略分析：与现有栈的整合

**与 Hermes Agent 的关联：**
- Hermes 已有 `native-mcp` skill（MCP Client 能力），本文深入解释了 MCP Server 端的企业级治理——什么该在 Server 端做（认证、Schema）、什么该在 Gateway 层做（权限、审计）
- Tool Registry 的设计思路可复用到 Hermes Skills 的发现与调用机制
- Credential Broker 模式（短期凭证注入）正是 Hermes gateway 中 `memory-tencentdb` 等服务的凭证管理可借鉴的方向

**核心启示：** 接入 MCP 解决"怎么连"，但真正让 Agent 在业务场景安全落地的是工具治理体系。当前市场工具（包括 Hermes）停留在"连接能跑"阶段，距离本文描述的五阶段成熟度还有很大差距——这也意味着巨大的工程机会。

---

*下一篇：从 Prompt Engineering 到 Context Engineering：Agent 如何设计模型每一步看到的世界*
