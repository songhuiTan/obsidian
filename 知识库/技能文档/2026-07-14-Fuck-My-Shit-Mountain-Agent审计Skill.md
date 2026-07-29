# Fuck My Shit Mountain：专治「项目能跑，但总觉得屎山里埋了东西」

> 来源：Github开源项目（微信公众号）
> 日期：2026-07-14
> 链接：https://mp.weixin.qq.com/s/yodRO-EoL2VbgxnCNdwORA
> GitHub：XiNian-dada/Fuck_My_Shit_Mountain

---

Coding Agent 审计 Skill — 让 Codex、Claude Code、Copilot、Gemini 等 Agent 对仓库进行全面审计。

## 核心功能

- Agent 先摸清仓库全貌，再按安全、稳定性、性能、测试、发布、可观测性等方向出报告
- **full 模式**：一口气查 **25 个维度**
- 每条发现包含：严重程度、置信度、代码证据、影响、修复建议和回归测试
- **诚实评分**：明确写每个维度看了什么、没看什么，覆盖置信度标为 High 或 Not assessed；没扫到证据不会顺手打"优秀"
- 输出 **Markdown + HTML** 报告（评分面板、风险表、覆盖矩阵、修复顺序）

## 安装

```
git clone [repo]
将 skill 目录复制进对应 Agent 的 skills 目录
```

## 注意

- AI 审计**不能替代**人工 review、测试和真实运行数据
- 老项目最难抓的不只是代码：CI 变量、定时任务、线上流量引起的超时
- 适合**上线前开会**或**接手老项目**时先扫一遍摸底

## 用途

那种"能跑，谁都不敢动"的仓库，先让它扒一遍，至少知道该先翻哪块垃圾。
