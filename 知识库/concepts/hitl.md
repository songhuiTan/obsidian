---
title: "HITL（Human-in-the-Loop，人工介入）"
created: 2026-07-21
updated: 2026-07-21
type: concept
tags: [agent-framework, production, enterprise, methodology]
sources:
  - "技能文档/2026-07-16-Agent治理-用Hook堵住LLM的偷懒越权与失忆.md"
  - "技能文档/2026-07-15-拆完WorkBuddy-我看到了生产级Agent的完整形态-叶小钗.md"
  - "技能文档/2026-07-06-从Vibe-Coding到Harness-腾讯大仓AI工程化实战.md"
  - "技能文档/2026-07-13-Loop工程不值钱-inspector才值钱.md"
  - "技能文档/2026-07-07-深入理解Deep-Agents-Subagents子代理编排机制.md"
confidence: medium
---

# HITL（Human-in-the-Loop）

HITL（Human-in-the-Loop）是指 Agent 执行过程中的人工介入机制，是生产级 Agent 系统的核心治理手段。在 prompt 层面约束不住的场景下，通过框架层的代码级强制兜底。

## 三类催生 HITL 的生产问题

| 问题 | 表现 | 根因 |
|------|------|------|
| 偷懒 | 长 SQL 截断、占位略写、复印重写到 token 耗尽 | 物理 token 预算不足 |
| 越权 | 用户没确认就调发布工具，推送未审核内容 | 模型无法区分操作的可逆性差异 |
| 失忆 | 改了表不分析下游风险，产出不通知用户 | 模型追求最短路径 |

## HITL 的实现机制

### Hook 链护栏
在 Agent 框架的关键切面挂载护栏逻辑：

- **beforeTool**：工具执行前，实现 HITL 门禁和危险操作确认
- **afterTool**：工具执行后，实现长文本 offload 和结果验证
- **before/afterModel**：每次 LLM 调用前后，支持用户取消
- **before/afterAgent**：Agent 运行前后，实现对话持久化

### 配置驱动
危险操作清单通过 YAML 配置，每个工具配授权标记和确认对话框。[[deep-agents]] 支持在 SubAgent 级别配置 interrupt_on，为特定工具设置 HITL 门禁。

## 从 Loop 工程到 Inspector

行业趋势从"Loop 工程不值钱"到"inspector 才值钱"：Agent 的循环执行机制已高度成熟，真正的差异化在于质检层和人工介入设计。

## 企业实践

**DECO（腾讯）**：通过 Hook 切面实现长文本完整性护栏（治偷懒）和危险操作确认（封越权）。

**WorkBuddy**：生产级 Agent 的完整形态包含分层 HITL 机制。

**Harness 工程化**：从 Vibe-Coding 到 Harness 的演进中，HITL 是确保 AI 安全落地的关键环节。

## 与 [[subagent-pattern]] 的关系

SubAgent 模式下每个子代理可独立配置 HITL 策略，实现细粒度的人工介入控制。[[hermes-agent]] 的 Hook 机制也借鉴了类似的切面设计。
