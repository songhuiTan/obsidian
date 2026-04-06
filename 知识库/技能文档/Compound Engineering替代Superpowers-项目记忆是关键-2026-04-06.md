# 为什么有人开始用 CE 替代 Superpowers？关键不只是流程，而是"记忆"

> 来源：微信公众号「探索与发现-Ultra」| 2026年4月2日
> 原文作者：Jason Zuo | X链接：https://x.com/xxxjzuo/status/2038086450013495554
> 归档时间：2026-04-06

## 核心观点

AI Agent 工作流接下来真正拉开差距的，不只是 planning、execution 和 review，而是有没有把"这次做事学到的东西"沉淀成下次还能复用的**项目记忆**。

## 1. Anthropic 的 harness 框架

Anthropic 工程博客提出的四个关键角色：
1. **Planner agent**：把大任务拆成 feature list
2. **Coding agent**：一次只做一个 feature
3. **Evaluator agent**：独立审查，不让 builder 给自己打分
4. **跨 session 桥接**：把上下文从一个 session 传到下一个 session

关键：**generator 和 evaluator 分开，效果会明显提升**。

## 2. gstack 的强项

- `Planner` + `浏览器端 Evaluator`
- `/plan-ceo-review`、`/plan-eng-review`：从产品和架构两个层面给需求把关
- `/qa` 打开浏览器去测 staging URL，像真实用户一样验证
- **更像一把锋利的刀，不是整套厨房**

## 3. Superpowers 为什么还不够

Superpowers 的历史地位：
- `brainstorm -> plan -> execute -> review` 帮很多人第一次从"跟 AI 瞎聊"升级到"有流程地用 AI"
- 120k stars
- 已做一定程度的 generator-evaluator 分离

CE 在三个层面做得更深：
- **Plan 更深**：`/ce:plan` 并行派出 research agents 搜项目历史经验、扫 codebase pattern、读 git history
- **Review 更细**：6-15 个专项 reviewer 并行（correctness、security、performance、testing、maintainability、adversarial）
- **能积累知识**：Superpowers 做完就完了，下一个 session 还是从零开始

## 4. CE 最值钱的：`/ce:compound`

每次做完功能或解决 bug，并行拉起三个 agent：
1. **Context Analyzer**
2. **Solution Extractor**
3. **Related Docs Finder**

写入 `docs/solutions/` 结构化知识：
- Problem
- What Didn't Work
- Solution
- Prevention

**核心区别：**
- Anthropic 的 progress file 解决的是**连续性**（"上一班交给下一班"）
- CE 的 `docs/solutions/` 解决的是**积累性**（"所有未来的班次都能查历史知识库"）

## 5. 逼近"永续型 Agent"

"永续"不是 24 小时工作，而是：
- 持续工作
- 持续沉淀
- 持续避免重复错误
- 持续减少重复浪费

## 6. compound 不应自动跑

不是每个 session 都值得沉淀（改 typo、调 CSS、跑 migration 不需要）。
提出折中方案：**compound janitor**，每天回头扫当天 session 和 git diff，只沉淀有价值的任务。

## 7. 最终组合：gstack + CE

- **gstack 负责**：`/plan-ceo-review`、`/plan-eng-review`、`/qa`（"做不做"和"真实测"）
- **CE 负责**：`/ce:plan`、`/ce:work`、`/ce:review`、`/ce:compound`（"怎么做""做得好不好"和"记住"）

一句话：**Superpowers 是很好的入门工作流，但 CE 的 compound 层是 Superpowers 完全没有补上的。**

## 标签

#ClaudeCode #Agent工作流 #CompoundEngineering #Superpowers #gstack #项目记忆
