# 我用 Matt Pocock 的 Teach Skill 学 Loop Engineering：一套任何人都能学会的学习方法论

> **来源**：微信公众号「观澜逸趣」(kunbo)
> **日期**：2026-06-28
> **链接**：https://mp.weixin.qq.com/s/CLVRJlPWsGrVxFXt4k7LPQ
> **标签**：`Matt Pocock` `Teach Skill` `Loop Engineering` `学习方法论` `认知科学` `反馈循环`
> **系列**：Loop Engineering 学习实践

---

## 核心问题

为什么看了 100 个教程还是什么都没学会？

**不是你不努力，而是你的学习方法本身就是错的。**

传统学习模式：看到感兴趣 → 买书/收藏 → 开始读 → 热情消退 → 放弃。只有输入没有输出，只有消费没有创造。

---

## /teach skill 的核心理念

### 三层学习路径

```
Knowledge（知识）→ Skills（技能）→ Wisdom（智慧）
```

- **Knowledge**：你知道什么（理解了概念）
- **Skills**：你能做什么（把"知道"变成"做到"）
- **Wisdom**：你什么时候知道该做什么（真实世界的判断）

> 大多数人只停留在 Knowledge，从未走到 Skills，更谈不上 Wisdom。

### Fluency vs Storage Strength

- **Fluency（流利度）**：你"当下"能不能想起来
- **Storage Strength（存储强度）**：是否能在大脑里长期留存

**流利度会给人虚假的掌握感**——刚看完教程能写出代码，不代表真的学会了。

### 记忆三技巧（认知科学验证）

| 技巧 | 做法 | 原理 |
|------|------|------|
| **Retrieval Practice（检索练习）** | 合上书本凭记忆回想 | 越费力回忆，记忆越牢固 |
| **Spacing（间隔重复）** | 分散时间学习（每天30min vs 周末3h） | 利用遗忘曲线，快忘时复习最有效 |
| **Interleaving（交错学习）** | 混着练不同类型 | 每道题要先判断类型再解题，训练更强 |

---

## 用 /teach 学 Loop Engineering：12 步完整演示

### Step 1：写 MISSION.md（目标驱动）

传统问题："我想学 Loop Engineering"——太模糊。

**Matt Pocock 的方法**：回答"我为什么要学这个？"

MISSION 模板要素：Why → Success looks like → Constraints → Out of scope

Mission 不是"我想学 X"，而是"学完后我能做 Y，改变 Z"。

### Step 2：评估起点（Zone of Proximal Development）

先搞清楚现在在哪，再决定下一步去哪。

Learning Record 记录两方面：
1. **已掌握的知识**（0001-established-prior-knowledge）
2. **已纠正的误解**（0002-corrected-misconception）

**Zone of Proximal Development（最近发展区）**：
- 🟢 舒适区：已经会的（不浪费时间重复）
- 🟡 最近发展区：跳一跳够得着（**应该学的**）
- 🔴 恐慌区：太难了（先放一放）

每一课都应该落在最近发展区里。

### Step 3-9：Knowledge + Skills 交替循环

Matt Pocock 的设计原则——**各层交替**：

| 课次 | 类型 | 主题 | 时长 |
|------|------|------|------|
| 第1课 | Knowledge | Loop Engineering 是什么？ | 15min |
| 第2课 | Skills（练习） | 画第一个 loop 概念图 | 30min |
| 第3课 | Knowledge | 五大原语（调度/Worktree/Skills/MCP/子Agent） | 25min |
| 第4课 | Skills（练习） | 写第一个 SKILL.md | 20min |
| 第5课 | Knowledge | 安全分级 L1/L2/L3 | 20min |
| 第6课 | Skills（练习） | 设计 loop 的 L1/L2/L3 路线图 | 25min |
| 第7课 | Knowledge | 深层概念（意图债务/理解债务/认知投降） | 20min |
| 第8课 | Skills（练习） | 设计 loop 安全边界 | 20min |

**每节课的核心原则**：
- **短小**：工作记忆有限，不能超载
- **快速完成**：15-30min，给出一个具体"胜利"
- **反馈循环**：做完检查→修正→再做→再检查

**练习的 Feedback Loop 模式**：
```
做一遍 → 对照标准检查 → 修改 → 再检查 → 完成
```

### Step 10：Wisdom 积累（真实世界实践）

在没有风险的项目上部署 L1 loop，记录 Learning Record：

```
发现的问题：
- loop 生成的报告太长了，重点不突出
- 已关闭的 issue 被标记为"需要关注"

调整方向：
- 修改 triage skill，要求输出结构化 markdown
- 添加规则：已关闭的 issue 自动过滤
```

### Step 11：持续 Loop（螺旋上升）

真正的学习是**螺旋式**的：
- 第1轮：理解概念、画流程图、写 SKILL.md
- 第2轮：部署 L1、校准 triage、调整配置
- 第3轮：升级到 L2、设计 verifier、处理失败案例
- 第4轮：部署 L3、监控 token 消耗、优化效率

---

## 底层逻辑：为什么有效？

### Mission-Driven（目标驱动）
传统："你有什么我学什么" → 这套："我要什么我学什么"

Mission 像过滤器：所有内容都要回答"这对我达成目标有没有用？"

### Feedback Loop（反馈循环）
满足两个条件：
1. **即时**：做完马上知道对不对
2. **准确**：反馈是真实的、正确的

### 学习范式的转变

| 维度 | 传统学习 | Loop学习 |
|------|---------|---------|
| 模式 | 漏斗（信息→遗忘） | 循环（做→反馈→改→再做） |
| 状态 | 每次从零开始 | 记住 Mission/已掌握/误解/最近发展区 |
| 角色 | 被动消费（看书/看视频） | 主动创造（画图/写SKILL.md/部署真实Loop） |

> **创造比消费的记忆强度高 10 倍。**

---

## 可迁移到任何场景

### 学英语
- Mission：三个月后能和外国客户开视频会议
- 每课只学5个商务句型
- 用 AI 语音工具模拟会议，即时纠正语法发音

### 学吉他
- Mission：三个月后在聚会上弹唱《Hotel California》前奏
- 每课只练一个和弦切换
- 录视频回放对照教程

### 考研
- 先做真题标出舒适区/最近发展区/恐慌区
- 只针对最近发展区的知识点
- 每学一个知识点，立刻做5道相关题目

---

## 从今天开始

1. **选一个想学的东西**
2. **写 MISSION.md**：为什么学？学会后有什么变化？什么时候达到什么水平？
3. **评估起点**：已知道什么？有什么误解？
4. **设计第一个 Loop**：15 分钟小课 → 立刻动手练习 → 获得即时反馈
5. **记录 Learning Record**：今天学会了什么？还有什么没懂？下一步该学什么？

> "Build the loop. But build it like someone who intends to stay the learner, not just the person who reads once."
> —— Matt Pocock

---

**参考来源：**
- Matt Pocock /teach Skill：https://github.com/mattpocock/skills/tree/main/skills/productivity/teach
- Addy Osmani Loop Engineering：https://addyosmani.com/blog/loop-engineering/
- Cobus Greyling loop-engineering：https://github.com/cobusgreyling/loop-engineering
