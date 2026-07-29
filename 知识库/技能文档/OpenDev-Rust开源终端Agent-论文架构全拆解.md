---
source: https://mp.weixin.qq.com/s/hX91eAzWGeR9P7iN7RnTeQ
author: 橙研所
date: 2026-06-23 01:56
archived: 2026-06-25
tags: [OpenDev, Rust, Agent, Harness, 论文拆解, 上下文管理, 终端编程, AI Agent 工程化, 归档]
---

# 一个 Rust 写的开源 Claude Code，把内脏全画了出来

> **核心摘要：** 深度拆解论文《Building Effective AI Coding Agents for the Terminal》。OpenDev 是一个 Rust 写的开源终端编程 agent（可类比开源 Rust 版 Claude Code），论文用 18 张原图逐模块拆解其工程架构。核心主张：自主 agent 的可靠性，靠的是把安全做成冗余的工程层、把上下文当内存来管，而不是某个聪明的 prompt。

---

## 00 论文背景

- **论文：** 《Building Effective AI Coding Agents for the Terminal》（81 页）
- **作者：** Nghi D. Q. Bui
- **核心项目：** **OpenDev** — Rust 写的开源命令行编程 agent
- **定位：** 开源、Rust 版的 Claude Code（终端原生、自主跑长任务）
- **范式背景：** AI 编程助手从 IDE 内（reactive copilot）迁移到命令行（开发操作中枢）

## 01 四级层级：session → agent → workflow → LLM

- **Session**：可并发、互相隔离的工作单元
- **Agent**：一个 session 里有多个专门化 subagent
- **Workflow**：每个 agent 跑带类型的工作流 — Execution / Thinking / Compaction
- **LLM**：每个 workflow **独立绑定** 用户配置的模型

> 核心思路：**按 workflow 选模型** — 执行用快便宜的、思考用强的、压缩用更省的，cost/latency/capability 逐工作流权衡。

## 02 系统四层架构

1. **Entry & UI 层**：用户输入与三种入口
2. **Agent 层**：推理核心（harness 所在）
3. **Tool & Context 层**：工具调度 + 上下文管理（全文最重的一层）
4. **Persistence 层**：跨会话状态 — Config Manager + Session Manager + Provider Cache

## 03 安全：五层纵深防御

| 层级 | 防护手段 |
|------|---------|
| L1 Prompt 级护栏 | system prompt 安全策略、只读优先、危险操作前先解释 |
| L2 Schema 级工具限制 | 按 subagent 过滤可见工具、MCP 发现 gating |
| L3 运行时审批 | plan/全自动/半自动模式，权限可持久化 |
| L4 工具级校验 | 危险模式黑名单、陈旧读检测、输出截断、超时 |
| L5 生命周期钩子 | pre-tool 钩子可直接阻断、可改写参数 |

> **关键设计：五层各自独立，任何一层失灵不会让整条防线崩溃。**

## 04 Agent Harness：ReAct 六相循环 + 七个子系统

中央 ReAct 循环六个相位：**预检与压缩 → 思考 → 自我批判 → 动作 → 工具执行 → 后处理**

外围七个子系统：消息处理、状态管理、审批、持久化等。

**专门化 subagent 团队（最小必要工具原则）：**
- **Code-Explorer**：深度代码探索、架构分析
- **Planner**：只读 + 写计划文件
- **PR-Reviewer / Security-Reviewer**：代码评审/安全审计（带严重度·置信度打分）
- **Web-Clone / Web-Generator**：网页视觉复刻 / 从规格生成 React+TS+Tailwind
- **Project-Init**：分析代码库生成项目指令文件
- **Ask-User**：只走 UI 出多选问卷，不经 LLM

## 05 双模式：Plan（只读探路）+ Normal（全权动手）

- **Plan Mode（只读）**：spawn Planner subagent，探代码库、分析模式、产出结构化计划 — 全程不写文件
- **Normal Mode（全权）**：完整读写、执行权限，真正动手改

## 06 输入分流：斜杠命令 vs 自然语言

- **斜杠命令**：经命令分发器路由到 9 个注册的命令处理器之一，**直接改系统状态** — 不消耗 LLM
- **自然语言**：进入 agent，走 ReAct 整套流程

> **工程原则：能用确定性代码处理的，绝不交给模型。**

## 07 死循环检测（doom-loop）

同「工具 + 参数」指纹在滑动窗口（最近 20 次调用）里重复 3 次 → 告警 + 跳过本轮工具执行。agent "鬼打墙"时系统主动打断，而不是无限烧 token。

## 08 上下文管理：六个子系统（全文最重的一层）

按生命周期顺序：

1. **动态 system prompt 构建** — 按条件谓词 + 优先级运行时拼装，非写死
2. **双重记忆** — 跨会话经验
3. **工具发现** — 按需暴露工具（懒加载）
4. **提醒（reminder）** — 对抗指令淡出（约 40 轮后模型会把重复 system message 当背景忽略）
5. **压缩（compaction）** — 渐进清理旧观察，分级降级
6. **检索（retrieval）** — 把代码喂进来

### Prompt 组装子系统
每个 prompt section 带两样东西：**条件谓词** + **优先级**。运行时求值，活下来的按优先级升序排。好处：按环境裁剪 + 可缓存。

### Reminder（提醒）子系统
8 个事件探测器在 ReAct 每轮后检查状态，命中则从模板库取对应模板注入上下文。解决 **指令淡出（instruction fade-out）**：长对话中初始指令被模型当成背景噪音忽略。

### 上下文压缩：分级渐进降级

| 阈值 | 动作 |
|------|------|
| 70% | 告警 |
| 80% | 遮罩观察（保留最近 6 条全保真） |
| 85% | 过渡态 |
| 90% | 激进遮罩（保留最近 3 条全保真） |
| 99% | 完全压缩 |

其他常量：
- 工具输出卸载：>8000 字符转 scratch 文件，只留 ~500 字预览
- 单条结果上限：~300 tokens
- 摘要重生成：每 5 条消息一次
- 能力缓存 TTL：24 小时

## 09 记忆系统：跨会话累积经验

记忆以 "playbook bullet"（经验条目）形式存。打分器三个维度：**历史有效性 + 新近度 + 与当前 query 的语义相似度**。选中条目注入 generator system prompt，本轮新学到的沉淀回 playbook。

## 10 检索四层：用户 query → 拼好的 LLM API 调用

1. **检索工具**：抓原始代码工件（文件、符号、引用）
2. **Code-Explorer subagent**：**隔离上下文** 里编排多步搜索
3. **综合层**：筛选、排序、压缩
4. **组装层**：拼成最终 API 调用

> **点睛设计：第 2 层「隔离上下文做检索」— 探索噪音关在子上下文里，主线只拿结论。**

## 11 工具层

- schema 组装器三个来源：① 静态内置定义 ② 动态 MCP 工具 ③ subagent schema
- main agent 共 **35 个内置工具**
- **懒加载工具发现**：LLM 发 `search_tools` 自然语言查询 → 按关键词打分 → 返回 top 匹配

## 12 符号检索：LSP 封装成 agent 工具

四层：Agent 工具层（6 个符号工具）→ 符号检索器（统一 API）→ LSP 包装层（语言检测/server 池/格式转换）→ 底层语言服务器

## 13 实现常量表（Table 9）

**循环与可靠性：**
- 死循环阈值：同指纹重复 3 次 / 窗口 20 次
- 最大 nudge 次数：3
- 最大撤销历史：50 步

**并发：**
- 思考档位：4 档（OFF/LOW/MEDIUM/HIGH）
- subagent 迭代上限：15
- 最大并发工具：5
- 编辑模糊匹配：9 遍

---

## 一句话总结

> OpenDev 给「自主编程 agent」提供的不是一个聪明 prompt，而是一张可被复制的工程蓝图：**把安全做成五层冗余、把上下文当内存管理、把规划与执行分离、把脏活关进隔离子上下文。模型负责想，harness 负责让想能安全、连续、可控地落地。**
