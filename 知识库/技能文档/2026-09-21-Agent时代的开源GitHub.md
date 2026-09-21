---
title: "Agent 时代的开源 GitHub 来了。。。"
source: "苍何（微信公众号）"
source_url: "https://mp.weixin.qq.com/s/XpOC2SrGIKUsfHpbIhPaAQ"
date: "2026-09-21"
tags: [AI, Agent, 开源, AgentGit, 会话版本控制]
---

# Agent 时代的开源 GitHub 来了。。。

> 来源：苍何｜微信公众号 2026-09-21 07:08（第 602 篇原创）
> 原文：[Agent 时代的开源 GitHub 来了。。。](https://mp.weixin.qq.com/s/XpOC2SrGIKUsfHpbIhPaAQ)
> GitHub：[Einsia/agent-git](https://github.com/Einsia/agent-git)

![文章封面](../assets/2026-09-21-AgentGit/cover.jpg)

## 概述

苍何用 GPT-6 Astra 搓了个 3D 个人网站，但 Codex 额度消耗极快——两个 Pro 号的周额度相继耗尽，换 Agent 又要重新读项目、重新理解上下文。本文介绍的开源项目 **AgentGit**（CLI 名 `agit`）正是为解决"换 Agent 就要重新讲一遍背景"的痛点而生：它会记录 Agent 的上下文、工具调用和关键决策，Session 可以保存、交接（Handoff）、多人协作，甚至发布到公开 Hub 供他人复用。作者用"星空、精灵漫游、访客留言板、占座、足迹星图"等功能的接力开发演示了 Wesight → Codex → Mimo 三个 Agent 的接续协作全程。

一句话定位：**给 Agent 用的 GitHub——"Code is cheap, show me the talk: version control for agent sessions"**（仓库官方描述）。

![3D 个人网站成品效果](../assets/2026-09-21-AgentGit/img_001.png)

## 核心功能

AgentGit 把 git 的版本控制理念搬到 Agent 会话层：

| 功能 | 说明 |
|------|------|
| **Session 记录** | 自动记录 Agent 上下文、工具调用、关键决策；磁盘上原本是随时会被覆盖/压缩/清理的 JSONL 文件，agit 在其上加快照和版本 |
| **Handoff（交接）** | 把前一个 Agent 整理的需求、上下文和关键决策打包交给下一个 Agent 继续实现，无需重复解释项目背景 |
| **仓库托管** | Session 存入指定仓库，获得权限的 Agent 读完即知项目进度和待办 |
| **分享协作** | Session 可分享给他人，支持只读/编辑权限；拿到权限的人可让自己的 Agent 读取上下文、继续修改提交 |
| **Remote Control** | 几个人可以共同指挥同一个 Agent——作者朋友接入后给网站加了「占座」功能，作者实时看到修改过程 |
| **Hub** | 公开 Session 的聚合广场，可搜索别人做过的任务、踩过的坑、用过的提示词 |

![AgentGit 项目页](../assets/2026-09-21-AgentGit/img_002.png)

支持的 Agent（文章口径）：**Claude Code、Codex、OpenCode、OpenClaw、Hermes、WorkBuddy**。注：README 官方口径为 Claude Code / Codex / OpenCode / Cursor，Hermes 等为文章补充声称，以仓库实际支持为准。

![agent-git.com 官网与安装方式](../assets/2026-09-21-AgentGit/img_003.png)

安装极简（README 验证）：

```sh
npx -y create-agit                 # 一键安装 agit + 接线 skills/hooks/MCP
# 或
npm install -g @einsia/agent-git
```

预编译二进制（npm 平台子包自动匹配 os/cpu），无需 Rust 工具链。⚠️ 不要装 `@einsia/agentgit`（无连字符）——那是重写前的旧 CLI，协议不兼容。

## 实战演示：三个 Agent 接力开发 3D 网站

作者以自己的 3D 个人网站为试验田，完整演示跨 Agent 接力：

1. **Wesight 安装并头脑风暴**——把 AgentGit 官方安装指令交给 Wesight 自己完成安装，然后让它分析网站还能加什么功能（昼夜天气循环、NPC 漫游、季节皮肤等）。
2. **Codex 读 Session 接力实现**——把 Wesight 的对话保存到 AgentGit，Codex 读取 Session 后给出星空、流星实现方案并落地；场景开始跟随现实时间变化。
3. **额度耗尽换 Mimo**——Codex 额度见底后直接切 Mimo，Mimo 读完 Codex 留下的进度和决策直接续写访客留言板。
4. **朋友协作**——分享 Session 给朋友（编辑权限），朋友的 Agent 接入后给网站新增「喷泉广场占座」功能（小精灵占座会提示"这座有人坐了，换一张吧"），全程作者实时可见。
5. **Hub 复用**——在 Hub 搜到「足迹星图」案例，让 Codex 读取该公开 Session 作参考，直接给自己的网站复刻了同款功能；作者随后也把足迹星图的制作过程公开回 Hub。

![Wesight 分析出的功能方案列表](../assets/2026-09-21-AgentGit/img_005.png)

![Codex 给出的实现方案](../assets/2026-09-21-AgentGit/img_006.png)

![AgentGit 支持的 Agent 列表（文章口径含 Hermes）](../assets/2026-09-21-AgentGit/img_007.png)

![访客星空与流星效果](../assets/2026-09-21-AgentGit/img_008.png)

![额度见底后切换 Mimo](../assets/2026-09-21-AgentGit/img_009.png)

![Mimo 接手后的访客留言板](../assets/2026-09-21-AgentGit/img_010.png)

## 关键洞察

![Session 存入指定仓库](../assets/2026-09-21-AgentGit/img_011.png)

![仓库中保存的 Session 列表](../assets/2026-09-21-AgentGit/img_012.png)

![Session 分享权限设置（只读/编辑）](../assets/2026-09-21-AgentGit/img_013.png)

![朋友的 Agent 接入并实时修改（Remote Control）](../assets/2026-09-21-AgentGit/img_014.png)

- **Agent 时代，"过程"开始比结果更值钱**——看到一个惊艳的案例，比起结果，更有价值的是需求怎么提、哪里踩过坑、为什么这样改、想法怎么一步步落地。这些藏在聊天记录里的经验终于能被保存、复用。
- **换 Agent 不再像给新人重新入职培训**——以前每次换 Agent 都像新员工入职，现在更像"把工位、资料和待办直接交过去"，省掉大量重复解释和重新理解项目的成本。
- **经验开源的新形态**：代码可以开源，Agent 干活的经验（提示词、决策路径、踩坑记录）也可以开源。一个人的经验在不同 Agent 和项目之间流动。
- 作者感慨的点睛之笔：**"当这些过程可以分享、交接和接力，很多事情就不用每个人都从零踩一遍坑。"**

## 战略分析

![Hub 中搜索公开 Session 案例](../assets/2026-09-21-AgentGit/img_016.png)

![「足迹星图」参考 Session](../assets/2026-09-21-AgentGit/img_017.png)

![足迹星图成品效果](../assets/2026-09-21-AgentGit/img_018.png)

### 与现有工具链的对照

AgentGit 与现有体系（Hermes Agent + Obsidian 知识库 + TencentDB Memory + GBrain）高度相关——它解决的正是多 Agent 会话持久化/交接问题，且明确声称支持 Hermes：

| 维度 | AgentGit (agit) | Hermes Agent 生态 | Obsidian 知识库 | GBrain |
|------|-----------------|-------------------|-----------------|--------|
| 定位 | Agent 会话的版本控制与协作平台 | 个人 AI Agent 框架（CLI/网关/技能） | 个人知识库（markdown + git） | 知识库向量化检索层（PostgreSQL+pgvector） |
| 会话记录 | JSONL 快照 + 版本 + 远端 Hub | 会话历史（本地）+ TencentDB 四层记忆 | 手动/半自动归档 | git-to-brain 同步 |
| 交接（Handoff） | 核心能力：跨 Agent Session 打包交接 | 无原生跨 Agent 交接 | 靠归档文档间接沉淀 | 靠页面/分块共享 |
| 多人协作 | Remote Control + 只读/编辑权限 | 单用户 | git 协作 | 单用户 |
| 经验复用 | Hub 公开 Session 搜索 | 技能库（skills）沉淀经验 | 快速导航/索引目录 | query/think 多跳问答 |
| 安装 | npx 一键，Rust 二进制 | 独立部署 | 本地 | Bun+PostgreSQL+Ollama 栈 |

### 差距与借鉴点

- **能力差距**：Hermes 生态目前没有等价的"会话版本控制 + 跨 Agent 交接"层。多 Agent 协作时（如 Codex CLI、Claude Code 与 Hermes 并用），上下文交接靠人肉转述或文档搬运。AgentGit 若真支持 Hermes（文章口径），可作为会话层粘合剂直接接入现有工作流。
- **Hub 的"经验开源"模式值得借鉴**：把"任务怎么做成"的完整过程（提示词+决策+操作）公开可搜，与知识库"归档文章"互补——文章归档沉淀的是别人的经验，Agent Session 归档沉淀的是自己 Agent 的过程资产。
- **风险提示**：项目 2026-09-01 创建，157⭐，发布 20 天即获文章推广，早期项目成熟度待观察；服务端（AgentGit server）独立仓库部署，Hub 功能依赖其云端，自托管需额外成本。频控/反爬等边缘行为文档未覆盖。

### 整合可能性

- 装一个 agit 试运行：`npx -y create-agit`，验证对 Hermes 会话目录的识别（文章称支持，README 未列——需实测确认）。
- 若支持成立，跨 Agent 工作流（Hermes ↔ Codex ↔ Claude Code）的交接成本大幅下降，可替代当前"导出会话→手动归档"的环节。
- Hub 上的公开 Session 是新的学习素材来源，可纳入知识库归档流水线（类似本篇）。

## 归档日志

![作者公开足迹星图制作过程](../assets/2026-09-21-AgentGit/img_019.png)

![Hub 上公开的 Session，可被搜索复用](../assets/2026-09-21-AgentGit/img_020.png)

- 2026-09-21 归档。Firecrawl 抓取正文 + GitHub API 核实仓库（157⭐/16 fork/MIT/Rust，2026-09-01 创建，发布前 20 天开源，持续活跃）；README 补充安装细节与 `@einsia/agentgit` 旧包警告。
- 图片下载一波三折：微信 CDN 对脚本 UA 与 fetch() 通道频控（返回 243B 占位图，curl/浏览器 fetch/Firecrawl stealth 出口均被拦），最终以「文章页内手动交换 data-src→src 强制 `<img>` 通道加载 + CDP `Page.getResourceContent` 提取缓存字节」组合拳取回全部 21 张（含封面），字节数与先期成功的锚点逐一吻合。
