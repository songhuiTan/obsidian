# MyHarness 构建思路总结：从"说明书"到"引擎"

> 归档时间：2026-04-06
> 关联文档：gstack+CE组合工作流方案、gstack+CE组合工作流-Harness差距分析
> 代码位置：/Users/songhuitan/Documents/ai/myharness/

---

## 一、构建全流程回顾

### Step 1：研究两个框架

**输入**：用户提供的 Obsidian 笔记 + 两个 GitHub 仓库 URL

通过并行 Agent 研究获取了完整的框架信息：

| 框架 | 核心发现 |
|------|---------|
| **gstack** | 31+ skill，Sprint 流程 Think→Plan→Build→Review→Test→Ship→Reflect，角色化（CEO/Eng/Design/QA/CSO），真实 Chromium 浏览器 QA，Conductor 并行，8 个 AI Agent 支持 |
| **CE Plugin** | 27 Agent（14 审查 + 6 研究 + 3 设计 + 4 工作流），20 命令，13 Skill，2 MCP Server（Playwright + Context7），核心差异化是 `/ce:compound` 知识沉淀 |

**关键洞察**（来自 Jason Zuo 推文）：
- gstack = 锋利的手术刀（精准决策 + 真实测试）
- CE = 完整的知识管理系统（深度规划 + 多 agent 审查 + 知识沉淀）
- 两者没有重叠，组合覆盖了 Anthropic harness 架构的所有角色

### Step 2：编写组合工作流方案

产出 `gstack+CE组合工作流方案-2026-04-06.md`，包含：
- 对比表（哪个框架负责什么）
- 7 阶段 Sprint 流程图
- 40+ 命令速查表
- 安装配置指南
- 5 个最佳实践
- 3 个典型场景

**但这是一个"说明书"。**

### Step 3：差距分析

用 OpenHarness 的 10 子系统架构作为基准逐项对照，发现 5 个核心差距：

| 差距 | 问题 |
|------|------|
| CLAUDE.md | 只是命令列表，没有决策逻辑、质量关卡、错误恢复 |
| 自定义 Skill | 组合工作流本身没有对应的 SKILL.md |
| Hooks | 完全没有 PreToolUse/PostToolUse 生命周期拦截 |
| Session 桥接 | 无进度文件、无跨 session 交接机制 |
| 自动化脚本 | 全手动操作，没有 CI 集成 |

### Step 4：实施构建

创建 `myharness/` 目录，20 个文件，1772 行内容：

```
核心引擎：    CLAUDE.md（10 条规则 + 状态机 + 关卡 + 恢复协议）
工作流 Skill： sprint / hotfix / compound-janitor / harness-init
自定义命令：  /sprint / /hotfix / /compound-janitor / /sprint-status / /harness-init
自定义 Agent： sprint-planner（只读规划）/ quality-gate（JSON 关卡检查）
Hooks：       PreToolUse（注入状态）/ PostToolUse（记录日志）
配置：        settings.json（权限规则）+ hooks.json（生命周期）
进度文件：    current-sprint.md + session-bridge.md
自动化脚本：  harness-init.sh + compound-janitor.sh
```

---

## 二、五个关键设计决策

### 决策 1：CLAUDE.md 是引擎指令，不是配置文件

```
说明书风格（旧）：
  - /plan-ceo-review：从产品角度审查需求
  - /ce:plan：深度技术规划

引擎指令风格（新）：
  收到需求时，自动选择工作流：
  | 包含"新功能" → Full Sprint → /office-hours |
  | 包含"bug"    → Hotfix      → /investigate  |
```

**为什么**：Claude Code 的 CLAUDE.md 在每次 session 开始时自动加载。它不是给人看的文档，是给 AI 执行的指令。必须编码"在什么条件下做什么"，而不只是"有什么可以用"。

### 决策 2：状态机 + 质量关卡

```
THINKING → PLANNING → BUILDING → REVIEWING → TESTING → SHIPPING → COMPOUNDING
              │            │           │          │
           GATE 1       GATE 2      GATE 3     GATE 4
         (审查通过)   (P0 清零)   (QA 通过)  (安全审查)
```

**为什么**：OpenHarness 的 engine 子系统展示了 Agent Loop 的核心——每次工具调用前检查权限、执行后记录结果。我们把同样模式应用到工作流层面：每个阶段转换前检查条件、完成后记录状态。

**为什么不可回退**：如果允许从 BUILDING 回到 THINKING，会导致无限循环。更好的策略是在当前阶段内解决问题，或者中止并创建新 Sprint。

### 决策 3：Session 桥接协议

```
每次新 session 开始：
  1. 读 current-sprint.md → 知道在哪个阶段
  2. 读 session-bridge.md → 知道上次做到哪
  3. 搜索 docs/solutions/ → 知道历史经验
  4. 向用户确认 → "上次做到 [X]，继续？"

每次 session 结束：
  1. 更新 current-sprint.md → 保存状态
  2. 覆写 session-bridge.md → 交接上下文
  3. 判断是否 compound → 沉淀知识
  4. 简要汇报 → 确认下次起点
```

**为什么**：Claude Code 的核心问题是"每次 session 从零开始"。OpenHarness 用 MEMORY.md 解决了跨 session 知识持久化，但没有解决"做到一半的工作怎么接续"。我们设计了两个文件各司其职：
- `current-sprint.md` = 持久状态（整个 Sprint 生命周期内持续更新）
- `session-bridge.md` = 交接文件（每次 session 结束覆盖写入）

### 决策 4：Compound Janitor 筛选机制

```
不是每个 session 都值得 compound：
  ✅ 新功能 2h+     → 架构决策
  ✅ 复杂 bug 修复   → 根因分析
  ✅ 性能优化       → 策略和基准
  ❌ 改 typo        → 低价值
  ❌ 调 CSS         → 低价值
  ❌ 跑 migration   → 标准操作
```

**为什么**：Jason Zuo 在推文中指出 CE 的 `/lfg` 全自动模式没有 compound 步骤。这是正确的——自动 compound 每个 session 会产生噪音，知识库被低价值内容淹没，反而降低 learnings-researcher 的搜索质量。

Janitor 的工作方式：
1. 扫描当日 git diff
2. 按变更量和类型自动评分（⭐⭐⭐/⭐⭐/⭐）
3. 展示候选列表供用户确认
4. 只对确认的条目执行 compound

### 决策 5：Skill 编码完整工作流

每个 SKILL.md 不是简单的命令列表，而是完整的工作流文档：
- 触发条件（什么时候用这个 skill）
- 前置检查（运行前验证什么）
- 每阶段的详细步骤和决策树
- 完成标志（怎么算做完了）
- 错误恢复策略（失败了怎么办）

**为什么**：Claude Code 的 Skill 系统是"按需知识加载"——只在需要时加载到 context。如果 Skill 只是命令别名，加载了也没有指导意义。完整的工作流文档让 Claude Code 在执行时有据可依。

---

## 三、从 OpenHarness 学到了什么

OpenHarness 的代码库是一个完整的"Agent 基础设施教科书"。关键学习：

| OpenHarness 设计 | 我们的借鉴 |
|-----------------|-----------|
| **10 子系统架构** | 用作差距分析的基准框架 |
| **Agent Loop** (`while True: stream → tool → loop`) | 设计了 Sprint 状态机（7 态单向流转） |
| **Hook 系统**（4 种类型：command/prompt/http/agent） | 设计了 PreToolUse + PostToolUse hooks |
| **Permission 系统**（3 模式 + path rules + command deny） | 设计了 settings.json（保护 .env/密钥/node_modules） |
| **Memory 系统**（MEMORY.md + 按项目 hash 存储 + 启发式搜索） | 设计了双层：progress 文件 + solutions 知识库 |
| **Skills 系统**（3 层加载：bundled → user → plugin） | 4 个自定义 skill（sprint/hotfix/janitor/init） |
| **Plugin 系统**（兼容 claude-code plugins） | 兼容性设计，myharness 可同时用于 Claude Code 和 OpenHarness |

---

## 四、最终产出物

```
myharness/                           20 个文件
├── CLAUDE.md                        引擎指令（240 行）
├── settings.json                    权限配置（24 行）
├── README.md                        项目文档
├── .claude/
│   ├── skills/（4 个）               Sprint/Hotfix/Janitor/Init
│   ├── commands/（5 个）             /sprint /hotfix /compound-janitor /sprint-status /harness-init
│   ├── agents/（2 个）               sprint-planner / quality-gate
│   └── hooks/                        hooks.json
├── docs/
│   ├── progress/（2 个模板）         current-sprint.md / session-bridge.md
│   └── solutions/（10 个分类 + patterns）
└── scripts/（2 个）                  harness-init.sh / compound-janitor.sh
```

**设计原则**：
1. 引擎而非说明书
2. 关卡不可跳过
3. 跨 Session 连续
4. 知识复利（不是每次都从零开始）
5. 渐进增强（按需启用更高级功能）

---

## 五、OpenSpace 集成：从"引擎"到"自进化"

### 5.1 OpenSpace 的核心发现

在完成第一版 MyHarness 后，我们研究了 OpenSpace (HKUDS) 的源码。OpenSpace 不是 harness，而是一个**自进化 Skill 引擎**，它提出了三个我们没想到的问题：

1. **Skill 应该是活的** — SKILL.md 为什么写好了就不变？
2. **每次使用都应产生反馈** — skill 被选中但没起作用，应该修复
3. **好的模式应该被捕获** — 发现可复用模式？变成新 skill

### 5.2 从 OpenSpace 借鉴的机制

| 机制 | OpenSpace 原理 | MyHarness 实现 |
|------|---------------|---------------|
| **三种进化模式** | FIX（修复）/ DERIVED（派生）/ CAPTURED（捕获） | `skill-evolution` Skill |
| **执行分析** | 每次执行后分析 skill 是否被正确应用 | `skill-metrics.md` 质量跟踪 |
| **技能谱系** | 父子关系、版本链、content diff | `.lineage.json` 侧文件 |
| **质量指标** | applied_rate / completion_rate / effective_rate | 4 个核心指标 + 评级标准 |
| **安全规则** | regex 扫描恶意模式 | `safety-check` Skill |
| **混合搜索** | BM25 + Embedding 排名 | 暂不纳入（无原生 embedding 支持） |

### 5.3 新增的文件（6 个）

```
.claude/skills/skill-evolution/SKILL.md   ← 自进化引擎（5 Phase）
.claude/skills/safety-check/SKILL.md      ← 安全审查（4 Phase）
.claude/commands/skill-evolution.md       ← /skill-evolution 命令
.claude/commands/safety-check.md          ← /safety-check 命令
docs/progress/skill-metrics.md            ← 技能质量跟踪
CLAUDE.md 新增 §11 自进化协议 + §12 双层知识体系
```

### 5.4 三层知识体系

```
第一层：问题知识（CE /ce:compound）
└── "N+1 查询的根因是 lazy loading"
→ 存储：docs/solutions/

第二层：技能知识（/skill-evolution）
└── "sprint skill 有效率 62% → 85%，因为增加了超时策略"
→ 存储：skill-metrics.md + .lineage.json

第三层：项目知识（CLAUDE.md + Session 桥接）
└── "这个项目用 JWT 认证，上次做到 BUILDING 阶段"
→ 存储：CLAUDE.md + current-sprint.md + session-bridge.md
```

### 5.5 不纳入的 OpenSpace 特性

| 特性 | 原因 |
|------|------|
| Python 后端引擎 | MyHarness 是 Claude Code 原生 |
| GUI 自动化 | Harness 不需要桌面操作 |
| 云端技能分享 | 过于复杂，后续考虑 |
| 嵌入向量搜索 | Claude Code 没有原生 embedding 支持 |
| SQLite 存储 | Markdown 文件更简单透明 |

### 5.6 最终统计

```
myharness/  →  30 个文件，~3100 行
├── 7 个 Skill
├── 8 个 Command
├── 4 个 Agent
├── 1 个 Hooks 配置
├── 4 个进度文件
├── 2 个脚本
└── 核心引擎（CLAUDE.md 14 条规则）
```

---

## 六、oh-my-openagent 集成：从"引擎"到"并行编排"

### 6.1 oh-my-openagent 的核心发现

在完成 MyHarness v1.0 后，我们研究了 oh-my-openagent（oh-my-opencode）——一个大型 TypeScript 插件（1602 文件，~214k LOC），它提供了 11 个 Agent、52 个生命周期 Hook、26 个工具。它解决了一个我们没覆盖的关键问题：**并行执行**。

oh-my-openagent 的核心并行机制：

| 机制 | 功能 |
|------|------|
| BackgroundManager | 并行任务生命周期管理（创建→运行→完成/取消） |
| ConcurrencyManager | 按模型/提供者限制并发（默认 5 并行） |
| Circuit Breaker | 循环检测，连续相同工具调用 20 次自动取消 |
| Category Routing | 按任务类型路由到最优模型（visual-engineering→Gemini, ultrabrain→GPT-5.4） |
| Ultrawork 协议 | 探索→汇聚→规划→分发→验证 五阶段并行编排 |
| Fallback Retry | 模型失败时自动切换到备用模型 |
| Spawn Limits | 最大 3 层嵌套 + 50 个子任务预算 |
| Stale Task Detection | 30 分钟无活动自动取消 |

### 6.2 从 oh-my-openagent 借鉴的机制

| 机制 | oh-my-openagent 实现 | MyHarness 适配 |
|------|--------------------|--------------|
| **并行编排** | BackgroundManager + task() 工具 | `/ultrawork` Skill（5 Phase: DECOMPOSE→DISPATCH→GATHER→INTEGRATE→COMPOUND） |
| **并发控制** | ConcurrencyManager（slot-based async acquire/release） | CLAUDE.md §13 规则（默认 5 并行，Explore 不计入上限） |
| **熔断器** | LoopDetector（consecutiveThreshold: 20） | CLAUDE.md §13.3（5 次相同调用→取消，100 次→取消，10 分钟超时→取消） |
| **任务路由** | Category Resolver（6 类别→对应模型） | CLAUDE.md §13.4（7 类别→对应 Agent 类型） |
| **Spawn 限制** | depth: 3, descendants: 50 | CLAUDE.md §13.2（最大 3 层嵌套，单次 15 子任务） |
| **任务跟踪** | BackgroundManager 内部状态 | `docs/progress/parallel-tasks.md` |
| **并行规划** | Prometheus agent | `parallel-planner` Agent（DAG + Wave 划分） |
| **任务执行** | Sisyphus-Junior agent | `task-runner` Agent（隔离执行 + 标准输出格式） |

### 6.3 新增的文件（5 个）

```
.claude/skills/ultrawork/SKILL.md          ← 并行编排技能（5 Phase + 熔断器 + 路由规则）
.claude/commands/ultrawork.md             ← /ultrawork 命令
.claude/agents/task-runner.md             ← 并行任务执行器
.claude/agents/parallel-planner.md        ← 并行任务规划器（DAG + Wave 划分）
docs/progress/parallel-tasks.md           ← 并行任务状态跟踪
```

CLAUDE.md 新增 §13 并行执行协议 + §14 Ultrawork 编排模式。

### 6.4 不纳入的 oh-my-openagent 特性

| 特性 | 原因 |
|------|------|
| 多模型路由（GPT/Gemini） | MyHarness 运行在 Claude Code 上，只有 Claude 模型 |
| tmux 视化面板 | Claude Code 没有 tmux 集成 |
| Fallback Retry Chain | 单模型不需要 fallback |
| Unstable Agent Babysitter | 不需要监控不稳定模型 |
| Python 后端引擎 | Claude Code 原生即可 |

### 6.5 四种并行模式

| 模式 | 适用场景 | Wave 结构 |
|------|---------|----------|
| Wave 并行 | 多个独立功能 | Wave 1: [A, B, C] |
| 两阶段 | 研究后实现 | Wave 1: [研究A, 研究B] → Wave 2: [实现A, 实现B] |
| 扇出审查 | 多文件审查 | Wave 1: [审文件1, 审文件2, 审文件3] |
| 竞争并行 | 方案对比 | Wave 1: [方案A, 方案B] → 选最优 |

### 6.6 构建历程总结

```
阶段 1：OpenHarness → 差距分析 → 引擎指令 + 状态机 + 关卡
阶段 2：OpenSpace   → 自进化   → Skill Evolution + 质量指标 + 谱系追踪
阶段 3：oh-my-openagent → 并行编排 → Ultrawork + 熔断器 + 任务路由

三个阶段的共同模式：
1. 先研究源码，理解核心机制
2. 识别与 MyHarness 的差距
3. 适配为 Claude Code 原生方案（不照搬 TypeScript 实现）
4. 整合到现有架构中（不破坏已有规则）
5. 更新所有相关文件（CLAUDE.md + README + Obsidian 笔记）
```

---

## 标签

#ClaudeCode #gstack #CompoundEngineering #OpenHarness #OpenSpace #oh-my-openagent #Harness #Agent工作流 #构建思路 #自进化 #并行编排
