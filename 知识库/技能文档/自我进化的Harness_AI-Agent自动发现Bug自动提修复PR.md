---
tags:
  - Agent
  - Harness工程
  - 自我进化
  - 自动修复
  - AI工程化
  - Tracing
source: 微信公众号 AI Agent 全栈技术分享
author: 谭光志
date: 2026-07-12
status: completed
---

# 让 AI Agent 系统自己发现 bug、自己提修复 PR：自我进化的 Harness

> 公众号 **AI Agent 全栈技术分享** · 谭光志 · 2026年6月29日
> Demo: [evo-agent-demo](https://github.com/woai3c/evo-agent-demo)（pnpm monorepo + Hono + React 19 + SQLite）

## 核心矛盾

**大多数 AI Agent 产品的 bug 出现在包裹模型的那层工程代码上——Harness。** 靠用户反馈发现问题太被动，手上没有出问题时的执行上下文。

> 模型是 CPU，Harness 是操作系统——操作系统管理进程调度和内存分配，Harness 管理工具调用、上下文窗口和错误恢复。

## 四步闭环：Tracing → 识别 → 修复 → 分析

### 第一步：Tracing（行车记录仪）

给 Agent 装上结构化日志，记录每一步的输入输出、耗时、token、错误。

**数据结构：** 一次 Operation（用户请求）包含多个 Step（LLM调用或工具调用）。

| 字段 | 说明 |
|------|------|
| type | call_llm / call_tool |
| duration_ms | 耗时 |
| tokens | input + output + cached |
| tool_name / tool_input / tool_output | 工具调用完整上下文 |
| error | 错误详情 |
| llm_response | 模型完整回复（供重放用） |

**设计要点：**
- **即时写入**——Agent 每执行完一步立刻写入数据库（SQLite WAL 模式），崩溃时已有前几步记录
- **完整保留 tool_input/tool_output**——管理面板可查看每一步输入输出
- **token 记录**——不同供应商字段名统一化（DeepSeek promptCacheHitTokens / Anthropic cacheReadInputTokens / OpenAI cachedPromptTokens）

### 第二步：错误 Pattern 自动识别

**分桶：** 未匹配错误按 provider × errorType × statusCode × toolName × message 做 GROUP BY 分组。同一种错误出现 15 次的桶只需分析一次。

**LLM 巡检：** 用 `generateObject` + Zod Schema 约束 LLM 输出结构。为每个桶生成 Pattern——包含名称、分类（user_error / provider_error / harness_bug / ignore）、匹配规则。

**回扫（Backfill）：** 新 Pattern 生成后自动回扫所有历史错误，把之前未匹配的标记上。

### 第三步：自动修复（系统自己提 PR）

对标记为 `harness_bug` 的 Pattern，启动修复 Agent：

1. 创建 git 分支（`fix/` 或 `improve/`）
2. 修复 Agent 配备文件工具（glob/grep/readFile/editFile/submitFix）
3. Agent 自主搜索相关文件、定位问题、应用修改
4. 有远程仓库则 push + 创建 PR，否则本地分支
5. 更新 `fix_status = 'pr_created'`

**安全边界：** 只生成 PR，不自动合并。合并仍需要开发者审核。

### 第四步：行为分析（没报错 ≠ 没问题）

**分析对象：** 所有操作（包括成功的），看三类"不健康"：
- 简单问题调了 8 次工具（效率低）
- 查询类操作成功率仅 65%（质量差）
- 搜索类 token 消耗是其他的 10 倍（成本高）

**流程：** 聚类（LLM 按意图分组）→ 打分（轻重任务不同阈值）→ 生成建议

| 维度 | 轻量任务（本地工具） | 重度任务（含网络工具） |
|------|-------------------|---------------------|
| 单次平均耗时 | ≤ 15 秒 | ≤ 60 秒 |
| 平均步数 | ≤ 10 | ≤ 20 |
| 平均 token | ≤ 50k | ≤ 150k |

健康分 < 0.8 或触发了 `low_success_rate` / `high_tool_error_rate` → 判定为"不健康"。critical 级别建议直接进入自动修复队列。

---

## 调度策略

| 任务 | 频率 | 理由 |
|------|------|------|
| 错误巡检（识别Pattern） | 每 1-2 小时 | 错误影响大，需快速发现 |
| 行为分析（健康度评估） | 每 24 小时 | 慢变量，需积累新数据 |
| 自动修复（生成PR） | 每天凌晨 | 代码修改需审慎，低峰期执行 |

## 重放引擎（验证修复是否生效）

用录好的 trace 数据替换真实 LLM + 工具，走同一份 `agentLoop()` 代码。被测试的是 Harness 拿到返回值后的处理逻辑（上下文管理、截断、错误恢复）。

**Safety Gate 流程：**
1. 未打补丁代码重放 → 必须失败（确认复现）
2. 已打补丁代码重放 → 必须成功（确认修复）
3. 回归验证 → 跑所有历史修复对应的 trace

## 生产效果（LobeHub 实践）

| 指标 | 初期 | 稳定后 |
|------|------|--------|
| Pattern 数量 | 31 条 | 104 条（9轮巡检后饱和） |
| 发现的 Harness 缺陷 | — | 20+ 个（Schema不兼容/负数max_tokens/reasoning_content丢失/Context Window过载等） |
| Agent 成功率 | ~75% | **95%+** |

## 成本估算（以 DeepSeek V4-Pro 为例）

| 任务 | 月 token 消耗 | 月成本 |
|------|-------------|--------|
| 巡检（720次/月） | ~720 万 | ~¥10 |
| 行为分析（30次/月） | ~60 万 | ~¥1 |
| 自动修复（10个目标/月） | 200-1000 万 | ~¥20-70 |
| **合计** | | **~¥30-80/月** |

## 自进化的三个层次

| 层次 | 人做什么 | 系统做什么 |
|------|---------|-----------|
| L1 纯人工 | 看日志、分类、修代码 | 无 |
| L2 辅助 | 确认分类 + 决定是否修 | 自动采集、找出可疑错误 |
| **L3 主导** | **审查 PR + 高风险决策** | **采集→识别→分类→生成修改→提PR** |

> 最重要的第一步：**加 Tracing。** 没有 trace 数据，后面什么都干不了。

---

## 参考

- 微信公众号：AI Agent 全栈技术分享（谭光志）
- 仓库：https://github.com/woai3c/evo-agent-demo
- 参考文章：《需要自进化的不是 Agent，而是 Harness》（LobeHub 生产实践）
- Vercel AI SDK
