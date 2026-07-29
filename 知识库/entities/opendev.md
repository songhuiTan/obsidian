---
title: OpenDev
created: 2026-07-21
updated: 2026-07-21
type: entity
tags: [tool, harness, skill-system, claude-code]
sources:
  - 技能文档/OpenDev-Rust开源终端Agent-论文架构全拆解.md
confidence: medium
---

# OpenDev

## 概述

OpenDev 是一个用 **Rust 编写的开源终端编程 Agent**，由 Nghi D. Q. Bui 开发，定位为"开源、Rust 版的 Claude Code"。其配套论文《Building Effective AI Coding Agents for the Terminal》长达 81 页，用 18 张原图逐模块拆解工程架构。核心主张：自主 agent 的可靠性，靠的是把安全做成冗余的工程层、把上下文当内存来管，而不是某个聪明的 prompt。

## 四级层级

- **Session** —— 可并发、互相隔离的工作单元
- **Agent** —— 一个 session 里有多个专门化 subagent
- **Workflow** —— 每个 agent 跑带类型的工作流（Execution / Thinking / Compaction）
- **LLM** —— 每个 workflow 独立绑定用户配置的模型

核心思路：按 workflow 选模型——执行用快便宜的、思考用强的、压缩用更省的，逐工作流权衡 cost/latency/capability。

## 四层系统架构

1. **Entry & UI 层** —— 用户输入与三种入口
2. **Agent 层** —— 推理核心（harness 所在）
3. **Tool & Context 层** —— 工具调度 + 上下文管理（全文最重的一层）
4. **Persistence 层** —— 跨会话状态（Config Manager + Session Manager + Provider Cache）

## 五层纵深防御

| 层级 | 防护手段 |
|------|---------|
| L1 Prompt 级护栏 | system prompt 安全策略、只读优先、危险操作前先解释 |
| L2 Schema 级工具限制 | 按 subagent 过滤可见工具、MCP 发现 gating |
| L3 运行时审批 | plan/全自动/半自动模式，权限可持久化 |
| L4 工具级校验 | 危险模式黑名单、陈旧读检测、输出截断、超时 |
| L5 生命周期钩子 | pre-tool 钩子可直接阻断、可改写参数 |

关键设计：五层各自独立，任何一层失灵不会让整条防线崩溃。

## Agent Harness：ReAct 六相循环 + 七个子系统

中央 ReAct 循环六个相位：预检与压缩 -> 思考 -> 自我批判 -> 动作 -> 工具执行 -> 后处理

专门化 subagent 团队（最小必要工具原则）：
- **Code-Explorer** —— 深度代码探索、架构分析
- **Planner** —— 只读 + 写计划文件
- **PR-Reviewer / Security-Reviewer** —— 代码评审/安全审计
- **Web-Clone / Web-Generator** —— 网页视觉复刻 / 从规格生成 React+TS+Tailwind
- **Project-Init** —— 分析代码库生成项目指令文件
- **Ask-User** —— 只走 UI 出多选问卷，不经 LLM

## 关键事实与日期

- 论文《Building Effective AI Coding Agents for the Terminal》81 页，18 张原图
- 双模式：Plan（只读探路）+ Normal（全权动手）
- 输入分流：斜杠命令经命令分发器路由到 9 个注册命令处理器，不消耗 LLM；自然语言走 ReAct 流程
- 死循环检测（doom-loop）：同"工具+参数"指纹在滑动窗口（最近 20 次调用）里重复 3 次 -> 告警 + 跳过本轮执行
- 工程原则：能用确定性代码处理的，绝不交给模型

## 相关文档

- [[Claude Code]] —— 闭源对标产品，OpenDev 可视为其开源 Rust 版
- [[CoreCoder]] —— 同样是 Claude Code 的开源复刻，但用 Python 实现，更偏教育
- [[Trellis]] —— 团队级 Agent Harness，OpenDev 可作为其底层执行引擎
- [[WorkBuddy]] —— 生产级 Agent 方法论，OpenDev 的安全防御设计符合生产级要求
- [[Hermes Agent]] —— 同为 Agent Harness 设计，不同语言和架构路径

## 与其他实体的关系

OpenDev 是 [[Claude Code]] 生态中重要的开源替代和架构参考。与 [[CoreCoder]]（Python 1,714 行教育级复刻）相比，OpenDev 是生产级 Rust 实现，工程完整性远超 CoreCoder。其五层纵深防御设计和专门化 subagent 团队理念，直接呼应 [[WorkBuddy]] 方法论中的"大脑系统""行动系统""控制系统"设计。

## 来源引用

- 技能文档/OpenDev-Rust开源终端Agent-论文架构全拆解.md
