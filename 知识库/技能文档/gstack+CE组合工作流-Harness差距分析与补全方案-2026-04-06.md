# gstack + CE 组合工作流 → Harness 差距分析与补全方案

> 归档时间：2026-04-06
> 基于 OpenHarness (HKUDS) 架构参考分析

---

## 一、现状诊断：你的文档是"说明书"，不是"引擎"

你的 `gstack+CE组合工作流方案-2026-04-06.md` 是一份优秀的**参考文档**，描述了：
- 哪个阶段用哪个命令
- 最佳实践和判断原则
- 场景示例

但 Claude Code 需要的不是说明书，而是**可执行的 harness 配置**——即一套 `.md` 文件、`settings.json`、hooks 规则、和目录结构，让 Claude Code **自动遵循**这个工作流。

**类比：** 说明书告诉你"开车先系安全带、再挂挡、踩油门"。Harness 是一个自动变速箱 + 安全锁——不系安全带就发动不了。

---

## 二、OpenHarness 的 Harness 架构（参考基准）

OpenHarness 定义了 Agent Harness 的 10 个子系统：

```
┌─────────────────────────────────────────────────┐
│                  Agent Harness                   │
├─────────────┬──────────────┬────────────────────┤
│  感知层      │  决策层       │  执行层            │
│             │              │                    │
│  prompts    │  engine      │  tools (43+)       │
│  (系统提示词) │  (Agent Loop)│  (文件/搜索/Web/MCP)│
│  CLAUDE.md  │              │                    │
│  skills     │  permissions │  tasks             │
│  (按需加载)  │  (安全边界)   │  (后台任务)         │
│             │              │                    │
│  memory     │  hooks       │  coordinator       │
│  (跨session) │  (生命周期)   │  (多Agent协调)      │
│             │              │                    │
│  config     │  commands    │  plugins           │
│  (配置)      │  (54个命令)   │  (扩展生态)         │
└─────────────┴──────────────┴────────────────────┘
```

**核心循环（Agent Loop）：**

```
while True:
    response = await api.stream(messages, tools)
    if response.stop_reason != "tool_use":
        break
    for tool_call in response.tool_uses:
        # Permission → Hook → Execute → Hook → Result
        result = await harness.execute_tool(tool_call)
    messages.append(tool_results)
```

---

## 三、差距分析：你的文档缺什么

### 对标 OpenHarness 10 子系统

| 子系统 | 你的文档 | 缺失 | 严重程度 |
|--------|---------|------|---------|
| **prompts** (CLAUDE.md) | 有一个简略模板 | 无决策树、无质量关卡、无错误处理流程 | 🔴 关键 |
| **engine** (Agent Loop) | 描述了流程但没有编码为自动化 | 无自动流转逻辑 | 🟡 中等 |
| **tools** | 列出了命令映射 | 缺少工具使用约束和顺序规则 | 🟡 中等 |
| **skills** (自定义) | 无 | 完全没有自定义 skill 文件 | 🔴 关键 |
| **plugins** | 描述了安装方式 | 无插件安装验证、无依赖检查 | 🟢 低 |
| **permissions** | 提到了 /careful /guard | 无 settings.json 权限规则 | 🟡 中等 |
| **hooks** | 无 | 完全没有 PreToolUse/PostToolUse hooks | 🔴 关键 |
| **memory** | 描述了双层记忆概念 | 无 MEMORY.md 结构、无 session 桥接文件 | 🔴 关键 |
| **commands** | 无 | 没有自定义工作流命令 | 🟡 中等 |
| **coordinator** | 提到了并行 Sprint | 无多 Agent 协调配置 | 🟡 中等 |

### 核心差距详解

#### 差距 1：没有可执行的 CLAUDE.md

**现状**：文档中的 CLAUDE.md 模板只是一个命令列表。
**需要**：完整的 CLAUDE.md 应该包含：
- 自动决策逻辑（"如果收到新功能需求，自动走 CEO Review 流程"）
- 质量关卡定义（"Eng Review 通过前不允许写代码"）
- 错误恢复流程（"如果 /qa 失败，回退到 /investigate"）
- 上下文注入规则（"每次 session 开始检查 docs/solutions/ 相关经验"）

#### 差距 2：没有自定义 Skill 文件

gstack 和 CE 各自有自己的 skill/command，但**组合工作流本身**没有对应的 skill 文件。需要：
- `sprint.md` — 完整 Sprint 工作流 skill
- `hotfix.md` — 紧急修复工作流 skill
- `compound-janitor.md` — 知识清理 skill
- `harness-init.md` — 项目初始化 skill

#### 差距 3：没有 Hooks 强制执行

没有 PreToolUse hooks 来**强制**工作流顺序。比如：
- Plan 审查未通过时，应该**阻止**代码修改
- 未运行 /qa 时，应该**阻止** /ship
- 敏感文件修改应该**触发**额外审查

#### 差距 4：没有 Session 桥接机制

Anthropic 的 harness 架构中，跨 session 桥接是核心。你的文档提到了 CE 的 `/ce:compound` 写知识库，但缺少：
- **Sprint 状态文件**：当前 Sprint 在哪个 Phase？上次做到哪？
- **Progress 文件**：session 之间的上下文传递
- **Resume 协议**：新 session 如何恢复上次工作

#### 差距 5：没有自动化脚本

手动跑命令不是 harness。harness 需要：
- 项目初始化脚本（创建目录结构、安装插件、生成配置）
- Sprint 启动脚本（自动创建 worktree、初始化计划文件）
- Compound Janitor 脚本（定时扫描 git diff、筛选有价值的 session）

---

## 四、补全方案：需要创建的文件清单

```
项目根目录/
├── CLAUDE.md                          ← 🔴 需要重写（完整工作流引擎）
├── .claude/
│   ├── skills/                        ← 🔴 需要创建（自定义 skill）
│   │   ├── sprint/
│   │   │   └── SKILL.md               ← 完整 Sprint 工作流
│   │   ├── hotfix/
│   │   │   └── SKILL.md               ← 紧急修复工作流
│   │   ├── compound-janitor/
│   │   │   └── SKILL.md               ← 知识清理工作流
│   │   └── harness-init/
│   │       └── SKILL.md               ← 项目初始化工作流
│   ├── commands/                      ← 🟡 可选（快捷命令）
│   │   ├── sprint.md                  ← /sprint 命令
│   │   ├── hotfix.md                  ← /hotfix 命令
│   │   └── compound-janitor.md        ← /compound-janitor 命令
│   ├── agents/                        ← 🟡 可选（自定义 agent）
│   │   ├── sprint-planner.md          ← Sprint 规划 agent
│   │   └── quality-gate.md            ← 质量关卡 agent
│   └── hooks/                         ← 🔴 需要创建
│       └── hooks.json                 ← PreToolUse/PostToolUse hooks
│
├── .claude-plugin/                    ← 🟡 可选（打包为插件）
│   └── plugin.json                    ← 插件清单
│
├── docs/
│   ├── solutions/                     ← CE 知识沉淀（已规划）
│   │   ├── patterns/
│   │   │   └── critical-patterns.md   ← 关键模式（需预创建）
│   │   └── (9个分类目录)
│   └── progress/                      ← 🔴 新增：Sprint 进度跟踪
│       ├── current-sprint.md          ← 当前 Sprint 状态
│       └── session-bridge.md          ← 跨 session 交接文件
├── plans/                             ← CE 计划文档（已规划）
│
├── settings.json                      ← 🟡 可选（权限和安全规则）
├── scripts/                           ← 🔴 新增：自动化脚本
│   ├── harness-init.sh                ← 项目初始化
│   └── compound-janitor.sh            ← 知识清理
└── MEMORY.md                          ← 🟡 可选（项目级记忆索引）
```

---

## 五、核心补全内容详解

### 5.1 CLAUDE.md 重写方案

当前的 CLAUDE.md 模板只是一个命令列表。需要重写为完整的"工作流引擎指令"：

```markdown
# 项目工作流 Harness

## 工作流引擎规则

### 自动决策逻辑
当收到需求时，按以下规则自动选择工作流：

1. **新功能/产品方向** → 触发 Full Sprint 流程
2. **技术改进/重构** → 触发 Tech Sprint 流程
3. **Bug 修复** → 触发 Hotfix 流程
4. **紧急线上问题** → 触发 Emergency 流程

### 质量关卡（不可跳过）
- GATE 1: Eng Review 通过前，禁止执行 /ce:work
- GATE 2: /ce:review P0 问题清零前，禁止执行 /qa
- GATE 3: /qa 通过前，禁止执行 /ship
- GATE 4: /ship 前必须运行 /cso（安全审查）

### Session 恢复协议
每次 session 开始时：
1. 读取 docs/progress/current-sprint.md 了解当前进度
2. 读取 docs/progress/session-bridge.md 获取上次交接
3. 用 learnings-researcher 检索 docs/solutions/ 相关经验
4. 向用户确认当前阶段和下一步

### 工作流详细规则
[每个工作流的详细步骤和决策树...]

### 知识沉淀规则
[Compound 的触发条件和筛选逻辑...]
```

### 5.2 Sprint Skill 文件

```markdown
---
name: sprint
description: 完整的 gstack+CE Sprint 工作流
  自动管理 THINK→PLAN→BUILD→REVIEW→TEST→SHIP→COMPOUND 全流程
---

# Sprint 工作流

## Phase 1: THINK
1. 执行 /office-hours 收集需求上下文
2. 写入 docs/progress/current-sprint.md

## Phase 2: PLAN
1. 执行 /autoplan（中小需求）或分步审查（大需求）
2. 审查通过后执行 /ce:plan
3. 确认 learnings-researcher 已检索历史经验
4. 更新 current-sprint.md 状态为 PLANNED

## Phase 3: BUILD
[详细执行规则...]

## Phase 4: REVIEW
[审查规则和关卡逻辑...]

## Phase 5: TEST
[测试规则...]

## Phase 6: SHIP
[部署规则...]

## Phase 7: COMPOUND
[知识沉淀规则...]
```

### 5.3 Hooks 配置

```json
{
  "hooks": {
    "session_start": [
      {
        "type": "command",
        "command": "cat docs/progress/current-sprint.md 2>/dev/null || echo 'No active sprint'"
      }
    ],
    "pre_tool_use": [
      {
        "type": "prompt",
        "matcher": "write|edit",
        "model": "haiku",
        "block_on_failure": true,
        "prompt": "检查：当前 Sprint 是否通过了 PLAN 阶段审查？如果 docs/progress/current-sprint.md 中 status 不是 PLANNED 或更后阶段，阻止写入操作并提示先完成审查。"
      }
    ],
    "post_tool_use": [
      {
        "type": "command",
        "matcher": "bash",
        "command": "echo '[$(date)] Tool used: $ARGUMENTS' >> docs/progress/session-bridge.md"
      }
    ]
  }
}
```

### 5.4 Sprint 状态文件 (current-sprint.md)

```markdown
---
sprint_id: sprint-2026-04-06-001
status: THINKING           # THINKING → PLANNING → BUILDING → REVIEWING → TESTING → SHIPPING → COMPOUNDING
created: 2026-04-06
updated: 2026-04-06
title: 用户认证功能
---

## 当前阶段
THINKING

## 已完成的步骤
- [ ] /office-hours
- [ ] /plan-ceo-review
- [ ] /plan-eng-review
- [ ] /ce:plan
- [ ] /ce:work
- [ ] /ce:review
- [ ] /review
- [ ] /qa
- [ ] /ship
- [ ] /ce:compound
- [ ] /retro

## 关键决策记录
（由 Claude Code 自动填写）

## 阻塞问题
（由 Claude Code 自动填写）
```

### 5.5 Session 桥接文件 (session-bridge.md)

```markdown
---
last_session: 2026-04-06T14:30:00
next_phase: BUILD
pending_items: []
---

## 上一班交接

### 完成的工作
- 完成了 /office-hours，确定了 MVP 范围
- 通过了 /autoplan 三轮审查
- 执行了 /ce:plan，生成 plans/user-auth.md

### 未完成的工作
- /ce:work 尚未开始
- plans/user-auth.md 中第 3 个 task 还需要确认

### 关键上下文
- 用户倾向使用 JWT 而非 session 认证
- 需要兼容移动端 API
- 参考 docs/solutions/auth/jwt-edge-runtime-20260315.md 的历史经验

### 下一步
1. 确认 plans/user-auth.md 第 3 个 task
2. 执行 /ce:work 开始开发
```

---

## 六、实施路线图

### Phase 1：最小可行 Harness（1-2 天）

只创建最核心的 3 个文件：

```
1. CLAUDE.md          — 工作流规则 + 决策逻辑 + Session 恢复协议
2. docs/progress/     — Sprint 状态 + Session 桥接
3. .claude/skills/    — sprint.md + hotfix.md
```

### Phase 2：质量关卡（2-3 天）

添加 hooks 和权限规则：

```
4. hooks.json         — PreToolUse 质量关卡
5. settings.json      — 权限和安全规则
6. compound-janitor   — 知识清理 skill
```

### Phase 3：自动化（3-5 天）

添加脚本和 CI 集成：

```
7. scripts/           — 初始化 + 清理脚本
8. .claude-plugin/    — 打包为可分享插件
9. CI/CD 集成         — GitHub Actions / 自动触发
```

---

## 七、与 OpenHarness 的兼容性

你的组合工作流可以通过两种方式运行：

| 方式 | 说明 | 适合场景 |
|------|------|---------|
| **Claude Code 原生** | 把 skill/hook/command 放到 `.claude/` 目录 | 日常开发、Claude Code 用户 |
| **OpenHarness** | 把同样的文件放到 `.openharness/` 目录 | 多模型后端、研究实验、非 Claude 模型 |

由于 OpenHarness 兼容 Claude Code 的 skill/plugin 格式，你创建的 skill 文件可以在两个平台上通用。

---

## 标签

#ClaudeCode #gstack #CompoundEngineering #OpenHarness #Agent工作流 #Harness #持续集成
