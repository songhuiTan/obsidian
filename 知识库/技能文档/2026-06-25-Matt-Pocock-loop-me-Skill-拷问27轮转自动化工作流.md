# Matt Pocock 发了新 loop-me Skill：拷问 27 轮后，它把你的日常工作变成了自动流程

> **来源**：Seebin 公众号  
> **日期**：2026-06-25  
> **原文链接**：https://mp.weixin.qq.com/s/9iOh0gG0Vp_gXip3FP6Xwg  
> **分类**：AI Agent 框架与工程化

---

## 文章概要

Matt Pocock 发布新 skill `loop-me`（仓库地址：https://github.com/mattpocock/skills），通过苏格拉底式盘问发现用户工作中的重复模式，输出 workflows/*.md 自动化工作流规格说明书。

---

## 核心内容

### Loop 透镜

- 提出「Loop 透镜」概念：一天、一周、一小时都可看作大 loop 套着小 loop
- 核心洞察：工作中有大量未被意识到的重复模式，可预测即值得委托

### 27 轮拷问

- 作者在 Codex 中运行 `/loop-me`，经历 20 分钟、27 轮层层递进的问题
- 生成两份文档：
  - `NOTES.md`：对用户工作模式的完整理解总结
  - `workflows/每日工作规划工作流.md`：完整的自动化流程定义
- 流程能识别工具链（Obsidian）、项目结构、精力分布模式（上午专注/下午沟通）

### 设计哲学：Push Right + Brief

- 把人介入点尽可能往后推
- Agent 完成所有能做的准备后，在最晚时刻提供决策就绪的简报
- 完成标准：一个 implementer agent 能直接照着 spec 来构建，不需要再问多余问题

### 与 grill-me 的区别

| | grill-me | loop-me |
|---|---|---|
| 核心动作 | 对齐已有计划 | 发现未抽象的模式 |
| 覆盖范围 | 决策分支全跑通 | 工作重复循环的规格沉淀 |
| 比喻 | 靶子校准 | 镜子照妖 |

### 真正的洞察

- 不是让 AI 直接接管，而是先帮用户想清楚「什么该接管」
- 先慢下来搞清楚自己的工作模式，再让 AI 自动化
- 当 loop 写成 spec 后：认知负荷下降、重复工作被委托、时间聚焦高价值决策

---

## 安装方式

```bash
# 安装全套技能
npx skills@latest add mattpocock/skills

# 或仅安装 loop-me
npx skills@latest add mattpocock/skills/in-progress/loop-me

# 初始化配置
/loop-me
```

---

## 关键词

`Matt Pocock` `loop-me` `grill-me` `Skill` `工作流自动化` `Codex` `Claude Code` `苏格拉底式盘问` `Push Right` `Brief` `Loop 透镜` `worfklows spec`
