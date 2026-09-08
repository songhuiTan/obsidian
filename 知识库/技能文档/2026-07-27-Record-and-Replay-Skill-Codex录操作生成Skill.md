# Codex 神级 Skill：一键把电脑流程操作整理成可复用 Skill

> 来源：nana娜娜呀｜抖音 2026-07-27
> 原文：https://www.douyin.com/video/7667401468241112497
> 标签：AI, Codex, Skill, 录屏, 自动化, 电脑操作, 教程

---

## 概述

nana娜娜呀 分享的 Codex 神级 Skill——**record-and-replay-skill**。可以一键录制电脑流程操作（鼠标点击、键盘输入等），自动整理成可复用的 Skill 文件，以后类似操作只需一句话即可让 AI 自动执行。

## 视频核心内容

视频演示了 Codex 的一个 Skill：记录你在电脑上的操作流程（如 AE 操作、发票整理、网站搭建等），自动生成 SKILL.md + 脚本，本质是把 RPA（机器人流程自动化）能力的门槛降到"录一遍"。

### Codex 的能力矩阵（视频一并展示）

| 能力 | 说明 |
|------|------|
| 📂 自动整理发票→Excel | 文件解析 + 结构化输出 |
| 🌐 计划模式做个人网站 | 需求→设计→实现 |
| 📑 一句话生成网页PPT | 录屏即生成 |
| ⚙️ Skill + 自动化 | 定时任务自动干活 |
| 🖥️ Computer Use | 直接接管鼠标操作 |

### 核心流程

```
录屏操作 → 分析步骤 → 生成 SKILL.md → 以后一句话复用
```

这个 Skill 的核心价值在于：**你不需要会写 Skill，只需要做一遍操作，Codex 帮你写成 Skill。**

## 社区反馈

评论区中有价值的观点：

- `"Codex插件官网原生自带的，自己安装就行"` — 说明该 Skill 来自 Codex 官方插件市场
- `"Token在燃烧"` — 录屏转 Skill 的 token 消耗较大
- `"对于一些复杂的 RPA，还是得按照传统的方式来"` — 复杂操作仍需传统 RPA
- `"如果你每天都有重复的工作，那么不建议 token燃烧"` — 常规高频重复任务用此方案不太划算
- `"windows用不了"` — 当前仅 macOS 支持

视频获 2068 点赞、2225 收藏、450 评论（数据截至归档）。

## 战略分析

"Record-and-Replay" 模式是 Skill 创作从"写"到"录"的范式转变。当前 Hermes/Codex/Claude Code 的 Skill 编写都是人工写 SKILL.md，而这个 Skill 让用户通过操作演示自动生成 Skill。

**对当前体系的启示：**

1. **降低 Skill 创作门槛** —— 当前知识库已归档大量关于"怎么写 Skill"的文章（歸藏 Skill 看法、Skill Quality Gate 等），如果 Skill 可以从操作演示自动生成，"写 Skill"将不再是门槛

2. **与 z-video-downloader 的协同** —— 刚刚安装的 z-video-downloader 是"下载视频"，这个 Skill 是"录操作生成 Skill"。两条路径在视频处理+Agent 自动化方向上互补

3. **Token 效率问题** —— 评论区反复提及 token 消耗大，说明录屏→Skill 的转换需要大量 LLM 调用。适合"一次录，反复用"的场景，不适合高频低价值的操作

4. **Computer Use 的延伸** —— 视频最后展示了 Computer Use 接管鼠标的能力。如果把 Record-and-Replay + Computer Use 结合起来，就是"先演示一次，以后让 AI 自己操作"的完整闭环

## 归档日志

- 2026-07-30 归档
（视频尝试下载，但抖音需要登录态，已通过页面提取全部关键信息归档）
