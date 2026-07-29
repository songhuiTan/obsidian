# 万字图文：Loop Engineering【附代码详解】

> 来源：阿杰Agent开发日志 (阿杰agent) · 2026-07-05
> 原文：https://mp.weixin.qq.com/s/ha0qgbjzFl6J_kE10HEVBg
> 分类：AI Agent 框架与工程化

---

> 三大工程（Prompt/Context/Harness）解决的是"这一次跑好"。Loop Engineering 解决的是"一直跑、跑好、跑完知道经验，出问题不瞎跑"。

---

## 一、四大工程的演进脉络

| 工程 | 核心问题 | 类比 |
|------|---------|------|
| **Prompt Engineering** | 怎么跟 AI 说话？ | 教员工说话 |
| **Context Engineering** | 给 AI 看什么信息？ | 配信息策展人 |
| **Harness Engineering** | AI 怎么安全运行？ | 建工作制度 |
| **Loop Engineering** | 系统怎么自动转？ | 设计工厂流水线 |

四大工程不是替代关系，是嵌套关系——缺任何一层，系统都会出问题。

---

## 二、思想根源：ReAct → Reflection → Loop

### ReAct（2022）
Reasoning + Acting 交替循环：Think → Act → Observe → Repeat。第一次把"思考"和"行动"编织成可迭代闭环。

| 对比 | Chain-of-Thought | ReAct |
|------|-----------------|-------|
| 与外界交互 | ❌ 纯内部推理 | ✅ 可以调工具 |
| 知识来源 | 模型内部 | 实时外部信息 |
| 容错 | 靠自己修正 | 观察到错误可重新思考 |
| 类比 | 闭卷考试 | 开卷考试+查资料 |

### Reflection（2023）
在 ReAct 基础上加"记忆"——失败后先复盘"为什么没成"，写进记忆，下轮带着经验来。**没有记忆的 Loop 是西西弗斯，有了记忆才是复利机器。**

### Loop Engineering
把 ReAct + Reflection 系统化、可控化、可审计地规模运转。

---

## 三、Agent Loop ≠ Loop Engineering

| 维度 | Agent Loop | Loop Engineering |
|------|-----------|-----------------|
| 本质 | 运行机制，一个功能 | 系统设计方法论 |
| 粒度 | 一条命令/一次递归 | 六大构件+状态 Schema+安全策略+成本模型 |
| 风险 | 可能空转、重复犯错、烧 token | 设计得当放大产能，设计不当放大错误 |

```python
# Agent Loop 是这个：
while not done:
    agent.run()

# Loop Engineering 是设计整个 main()：
def main():
    task = discover_task()
    state = load_state()
    env = setup_worktree(task)
    result = maker.run(task, state)
    verdict = checker.verify(result)
    if verdict.risk > threshold:
        escalate_to_human()
    save_state(result, verdict)
    update_budget(tokens_used)
```

---

## 四、六大构件

1. **Automations/Scheduling（心跳调度）**——主动发现任务、自动分诊
2. **Worktrees（隔离环境）**——每个 Agent 独立操作间，git worktree 思想并行不干扰
3. **Skills（项目知识库）**——还 Intent Debt（意图债），避免 Agent 每轮冷启动
4. **Plugins & Connectors（MCP 连接器）**——即插即用接入外部系统
5. **Sub-agents 分工：Maker/Checker 分离**——写代码的和验证的必须是不同 Agent，禁止自评
6. **Human Gate（人工闸门）**——高风险操作必须暂停等待人工审核

---

## 五、三笔"债"

- **Intent Debt（意图债）**：团队规范没写进 Skills，Agent 每轮在猜
- **Comprehension Debt（理解债）**：Loop 越快，没认真读过的代码越多
- **Cognitive Surrender（认知投降）**：把 Loop 当"不需要动脑子"的按钮

---

## 六、分阶段上线

| 等级 | 模式 | 目标 |
|------|------|------|
| **L1** | 只看报告，不行动 | 建立信任，观察判断准确性 |
| **L2** | 小步自动修复 + 独立 Verifier | 积累数据，发现盲区 |
| **L3** | 无人值守运行 | 护栏到位+多 Loop 协调+定期审计 |

判断标准：L1 运行两周，对每一条报告都认为"判断对了"，才考虑升 L2。

---

## 七、nanobot 源码解析：三层嵌套循环

nanobot (OpenClaw 生态) 的核心是**三层俄罗斯套娃**结构：

### 第一层：消息总线循环
`loop.py` 主循环：async 监听消息，`timeout=1.0` 空闲时做会话压缩检查，`asyncio.create_task` 非阻塞分发。

### 第二层：状态机循环
每条消息走 8 状态流程：RESTORE → COMPACT → COMMAND → BUILD → RUN → SAVE → RESPOND → DONE。通过转移表 `_TRANSITIONS` 驱动，每个状态职责单一。

### 第三层：工具迭代循环（核心）
`runner.py` `_run_core`：ReAct 的工程实现。两个分支：
- LLM 要调工具 → 执行 → continue（回到循环头）
- LLM 给出答案 → break（结束循环）

**关键工程细节：**
- **上下文治理**：`microcompact`（旧工具结果压缩单行）+ `snip_history`（超出窗口从头部裁剪，system 消息永远保留）
- **并发工具调度**：`concurrency_safe` 工具用 `asyncio.gather` 并行，否则串行隔离
- **mid-turn 注入**：后台 SubAgent 完成结果通过 pending_queue 实时注入主循环，对应 Reflection 模式

---

## 八、你现在在哪个阶段？

| 阶段 | 状态 |
|------|------|
| 手动写每轮 prompt | Prompt/Context 阶段，先固化 Skills |
| 会 `/loop` 但没有 STATE/Verifier | 有 Loop 无 Engineering，补 STATE.md + L1 |
| 有分诊 Skill + Maker/Checker 分离 | Loop Engineering 入门，跑 loop-audit，设 token budget |
| 多 Loop 并行 + 完整闸门 + 可观测性 | 成熟，定期审计输出质量 |

> 从写 prompt 到设计 loop，不是技能的替代，是视角的外移——你从在系统里工作，变成了设计系统。
