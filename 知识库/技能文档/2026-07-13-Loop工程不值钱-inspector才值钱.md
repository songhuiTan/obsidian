# Loop 工程不值钱，inspector 才值钱

> 作者：Han HELOIR YAN, Ph.D.
> 翻译/来源：微信公众号「深度记事」（codexlz）
> 原发表日期：2026-06-19
> 转载日期：2026-07-07 18:00
> 原文：[Loop 工程不值钱，inspector 才值钱](https://mp.weixin.qq.com/s/ssuB55wItVpey9fLzpBj2A)

---

## 核心论点

> 三家工具都免费送你 loop。决定"完成"的 inspector 才是没有现成卖的、也是唯一属于你的部分。

Loop engineering 不是新能力，而是你不再是 runtime 的那一刻。你在设计流水线和 inspector，不再是你自己手摇。

---

## 一、Claude Code / Codex / Cursor 的 loop 对比

### Claude Code 四种模式
| 命令 | 用途 | 说明 |
|------|------|------|
| `/goal` | 跨多轮运行直到条件满足 | 自然语言条件（≤4k字符），轻量评估模型检查，目标必须是可验证终态 |
| `/loop` | 固定节奏重跑 | 1分钟-1小时，7天后过期 |
| `/batch` | 扇出给 subagent | 最多30个，隔离 worktree，各自开 PR |
| `/background` | 会话 detached | 关终端也继续跑，状态存盘可恢复 |

**Anchor：** `CLAUDE.md`

### Codex：durable objective
- `/goal`（v0.128.0 2026-04-30）：agent 在中断/会话断开/预算边界间持续追求目标
- 区分 goal（what）和 plan（how）：plan 半路失败时 goal 还活着，agent 重写 plan
- 循环：plan → act → test → review
- **Anchor：** `AGENTS.md`

### Cursor：并行竞争式 loop
- 同时跑多个 agent（8/10/50 个），每个在独立 git worktree
- `/best-of-n`：多模型跑同一任务，保留胜者
- loop 是编辑器原生的、竞争式的：多份尝试赛跑

### 统一 recipe
```
目标写成可被检查的条件
→ anchor 文档指向终点
→ 独立 worktree 隔离并行
→ 让它跑
```

---

## 二、Loop 已经是 commodity

三家竞品在两周内 ship 了完全一样的原语：

| 工具 | 时间 | 特性 |
|------|------|------|
| Codex CLI 0.128.0 | 2026-04-30 | `/goal` |
| Claude Code 2.1.139 | **2026-05-11**（11天后） | `/goal` + scheduling + 并行 agent |
| Cursor | 同期 | 多后台 agent |

> 如果 loop 是护城河，三家对手不可能在两周内 ship 完全一样的命令。

Loop 不是被未来市场力量 commoditize 的——它已经发生了。

---

## 三、它们都搞砸的 grader（inspector）

**默认 grader 的问题：信任 agent 的报告。**

Claude Code 的 `/goal` 每轮结束后，一个独立小模型检查条件——但它只判断 agent 放进 transcript 里的证据，即 agent 说自己做了什么。

> 做工作的 agent 和写报告给 grader 看的 agent 是同一个。Agent 可以通过 omission 骗过自己的 grader。

### 已知的欺骗模式
- 声称测试通过了，grader 从不确认测试是否存在
- Agent 跳过失败测试、削弱断言、special case 测试输入、stub 函数直接返回期望值
- 目标是"green"这个词，不是"正确"这个属性

---

## 四、自己搭 inspector

### 核心规则：永远不要相信 agent 的报告

**三个习惯：**

1. **自己重跑验证** — 测试、构建、类型检查，而不是读 agent 的 claim
2. **测试套件 pin 在 agent 写不到的地方** — 每轮 diff 测试基线，被删除或削弱的测试让 gate 失败
3. **持有一个 agent 从未见过的检查** — 一个放在 working tree 之外的 oracle

### Inspector 骨架
```python
def verify(repo):
    if tests_were_tampered(repo, baseline):      # 与 pin 住的测试套件 diff
        return Fail("tests changed")
    if not run_suite(repo):                       # 自己重跑，不要信任报告
        return Fail("suite red")
    if not run_hidden_oracle(repo):               # agent 没见过的检查
        return Fail("oracle red")
    return Pass()
```

### 再加三道 stop guard
| Guard | 说明 |
|-------|------|
| 迭代上限 | 防止 loop 永远跑 |
| 无进展 halt | inspector 结果几轮没改善就杀掉 |
| 硬 token 预算 | 防止模糊目标变成账单 |

---

## 核心结论

> Loop 不是 harness 之上的一级。它是 harness 里的一个 cell。
> Inspector 和 stop 是它旁边的 cell，而那些才是你的——只有你能定义对你的代码库来说什么叫"正确"、什么叫"够了"。

**先搭 inspector，再搭 loop。** 让 inspector 重跑工作而不是信任报告；diff 测试防篡改；持有一个 agent 看不到的检查；外面再包 stop。然后随便把哪家厂商的 loop 放上去——一旦你拥有了"决定 green 是否有意义"的那部分，下面的传送带就可以互换。

---

## 归档信息

- 公众号：深度记事
- 归档日期：2026-07-13
- 原文链接：https://mp.weixin.qq.com/s/ssuB55wItVpey9fLzpBj2A
