# 60 分钟，成为 AI Native 组织：人、Agent 与上下文三层系统

> **来源：** 公众号「AI 启蒙小伙伴」@邵猛
> **发布时间：** 2026-06-21
> **原文：** https://mp.weixin.qq.com/s/me57WtxigTxFaMM8mL5tjQ
> **原始对谈：** Greg Isenberg × Theo Tabah — https://youtube.com/watch?v=LztPaNmcWGU

## 核心框架

> **AI Native 组织 = 人管理 agents + agents 读写公司 + 公司随时间变得更聪明。**

价值锚点（DeepMind CEO Demis Hassabis）：**"以 100 英里时速跑错方向，比站着不动更糟。"** 速度必须服务于客户与方向。

---

## 第一层：人 — 守住"起手"和"收尾"

**核心 reframe：每个人都是 manager。**

| 阶段 | AI 之前 | AI 之后 |
|------|---------|---------|
| 战略/判断（前端 bookend） | 占比较小 | **人聚焦于此** |
| 执行（middle） | 占大头 | **AI 吃掉** |
| 沟通/评审（后端 bookend） | 占比较小 | **人聚焦于此** |

管理者的成功 = 他团队（含 agent）的成功。把 agent 当成下属来 set up for success。

---

## 第二层：Agents — 给到四样东西，它才能"自治"

### Agent 成熟度三层

1. **基础层**：和 ChatGPT 聊天
2. **半自治层**：agent 在跑，但你不停点 approve
3. **自治层**：像新员工，前期 babysit，后期可独立运行数天甚至数周

### 达到自治的四个前提（类比"新人入职第一天"）

1. **Clear Goal** — 清晰目标
2. **Skills** — 技能（markdown 文件）
3. **Tools** — 工具
4. **Context** — 上下文

四者缺一，agent 就会失败。

### Eval：让"好"变得可见

对 agent 输出的可视化评估——来源于 skill 中的质量标准（SOP/标杆样本）+ goal 中的成功定义 + context 中的参照文档。把 eval 烤进系统，输出质量才能稳定复现。

### Skill Chains：本期最被低估的概念

- **Skill** = markdown 文件
- **Skill Chain** = 一个 macro skill 顺序触发多个 skill，像 playbook 串行执行

> AI 的幻觉就像一个急于表现的实习生"fake it till you make it"被放大了一千倍。Skill chain 通过 QA skill 反复查验，把幻觉压到最低。

---

## 第三层：Context — "让公司对 agent 可读"

**没有 context layer，前面的人和 agent 都无法真正自治。**

### 灵魂拷问

任何规模的组织里，人对自己公司都是"半盲"的。Context layer 的作用就是给 agent 一双 20/20 的完美视力。

### 上下文层的五阶段循环

```
Capture → Curate → Store → Execute → Experience →（回流 Capture）
```

1. **Capture（采集）**：从 Slack、邮件、会议录音、Linear 等定期拉取信息
2. **Curate（筛选）**：像图书管理员一样清洗、归档、判断哪些需要保留
3. **Store（存储）**：**"Brain" = 一堆文件夹 + markdown 文件**，agent 可搜索、检索、写回
4. **Execute（执行）**：agent 调用 context 完成工作
5. **Experience（体验）+ 回流**：客户使用产品 → 信号 → 回流进 brain → 系统变聪明

### Traces / Exhaust（痕迹/废气）

产品开发中的中间决策、探索文档、被砍掉的方案——通常躺在没人看的文件夹里腐烂。把它们也回流进 brain，未来 agent 可以基于"我们当时为什么这么决策"创造新东西。

### 人工闸门

Experience → Capture 这一环必须有人把关，否则错误 context 会污染整个 brain。

---

## Live Demo 1：提案微站（Proposal Skill Chain）

**场景**：Spotify 是潜在客户，系统自动扫描 transcript/邮件识别到 "proposal request" → 自动触发 skill chain。

**链路（3 个 skill 顺序执行）**：
1. **Build microsite** — 生成带双品牌的高保真提案微站
2. **Copy skill** — 确保语气像 Theo 本人，不像 AI
3. **QA skill** — 审查不夸大、不杜撰，所有内容源自真实记录

**关键结果**：
- 3–4 分钟生成完整可分享的微站，ping 到 Slack
- 微站内嵌了几个月前通话中的个性化细节（客户说过的话、个人趣事等）
- Theo 直言：这套系统已经为 LCA 带来**"数百万美元收入"**

---

## Live Demo 2：10 分钟内出可用产品 + 当场做用户测试

**Skill Chain（5 个 skill）**：
1. **Hypothesis** — 明确要验证的假设
2. **Build prototype**
3. **Usability test** — 内置用户测试问卷
4. **Feedback synthesis** — 把多份反馈合成 lessons
5. **V2** — 基于反馈当场出第二版

**时间对比**：

| 任务 | 传统 | AI Native |
|------|------|-----------|
| Proposal | 最多 3 天 | 几分钟 |
| 原型 + 收集反馈 + 合成 V2 | 1–2 周 | 一个 session |

### 如何 bootstrap context（给资源有限者）

- 用 **Mobbin MCP**（海量优秀 app 流程库）
- 抓取目标公司的 design system 做成 skill
- 找到对应 output 类型的 MCP
- 逐步把 context 灌进去——**不需要一开始就有全部 context**

---

## 创业方向：把三层系统产品化为服务

### 框架

沿三个向量 niche down：

1. **Industry** — 商业地产、牙科、餐饮等
2. **Function** — 服务哪个职能团队
3. **Company size** — 别太小（没预算）

### 优先级矩阵

| | 高频 | 低频 |
|---|------|------|
| **Niche** | ⭐ 先做（layup） | 后做 |
| **General** | 次做 | — |

找到 **niche + 高频** 的工作流，反复在销售、brief、提案、内容里展示。

---

## 三个深层判断

1. **AI Native 的本质不是"用 AI"，而是"建系统"。** 人/agent/context 三层缺一不可，context layer 是最被低估、也是真正的护城河来源。
2. **Skill Chain 是对抗幻觉、实现 agent 自治的关键机制。** 单个 skill 不够，必须把 build → copy → QA 串成 playbook，质量才能稳定。
3. **速度只有在能换回 signal 时才有意义。** 快速出原型、当场测用户、当场出 V2——闭环越短，护城河越深。

---

## 相关资源

- [Anthropic Claude 给创业者的 AI-Native 构建手册](https://mp.weixin.qq.com/s?__biz=MzkwNDExODE4Nw==&mid=2247496964&idx=1&sn=6726551d691d1a14071b97c4d899532b)
- [大前端 AI Native 开发三端基础设施](https://mp.weixin.qq.com/s?__biz=MzkwNDExODE4Nw==&mid=2247496777&idx=1&sn=a2a791a18aef2c60f5a34020a0fc6637)
- [OpenAI Codex 核心成员：AI Native 团队研发实践](https://mp.weixin.qq.com/s?__biz=MzkwNDExODE4Nw==&mid=2247496683&idx=1&sn=11d899f9a7b8c75c8d0c93209bd2cc72)

## 归档日志

- 2026-06-22 归档
