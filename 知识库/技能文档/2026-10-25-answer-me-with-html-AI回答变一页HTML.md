---
title: "又一个神级 Skill：把 AI 的几百行回答，变成一页 HTML · answer-me-with-html"
source: "芋道源码（AI探针）"
source_url: "https://mp.weixin.qq.com/s/ciASeS9ahRiievBaav-Nkw"
date: "2026-10-25"
tags: [answer-me-with-html, QingYunA, AgentSkill, ClaudeCode, Codex, DeepSeekHarness, HTML渲染, HTML渲染, 可视化, ASD-STE100, Karpathy, 微信公众号归档]
---

# 又一个神级 Skill：把 AI 的几百行回答，变成一页 HTML

> 归档日 2026-10-26 ｜ 原文发布 2026-10-25 21:30 ｜ 公众号「芋道源码」AI探针栏目

## 概述

`answer-me-with-html`（作者 QingYunA，MIT 许可）是一个 Agent Skill：装进 Claude Code / Codex / Cursor / OpenCode / Pi / DeepSeek Harness 后，遇到复杂问题，模型除了回文字，还会生成一页本地 HTML——里面有时序图、流程图、对比表和标注，浏览器打开即看，断网也能看。

核心分工只有一句：**内容归模型，排版归程序。**

仓库实时数据（2026-10-26 归档时查 GitHub API）：**4,435 ⭐ / 301 fork / JavaScript / MIT**，创建于 2026-10-02，最后推送 2026-10-26，仍在日更。原文说「开源两周多，Star 三千六百多」，一天之内涨了 800+。

## 核心亮点

### 1. 它解决的问题不是"AI 答得不清楚"，而是"人跟不上"

作者引的是 Karpathy 的判断：**AI 干的活越多，人跟不跟得上 AI 的输出反而成了难题。**

问一个架构问题，回答刷出好几百行。读到一半忘了前面，想回头核对某段配置又得往上翻——这不全是注意力问题，是**一维文本不适合承载多维信息**。同一份信息换成一页带图的面板，信息量没变，读法变了。

### 2. 为什么不让模型直接写 HTML——一笔 token 账

作者数了模型手写的 9 个页面，平均 **4,893 token**：

| 页面组成 | 占比 | 装上 skill 后 |
|---|---:|---|
| SVG 图的坐标与路径 | 47% | CLI 算 |
| CSS | 15% | CLI 写 |
| HTML 标签 | 17% | CLI 写 |
| 正文文本 | 21% | 模型写（Markdown 草稿） |

模型写了十成东西，只有两成是"话"，八成是架子。所以拆开：**模型只写几百 token 的 Markdown 草稿，排版、配色、画图交给 skill 自带 CLI**，草稿扔进去约 **50ms** 出页面。

### 3. 实测数字（Claude Sonnet 5.5，3 主题 × 3 次中位数）

| | 直接要 HTML | 装 answer-me-with-html | 差异 |
|---|---:|---:|---|
| 模型输出 token | 4,893 | **612** | **少 8×** |
| 耗时 | 31 s | **12 s** | **快 2.6×** |
| 实际账单 | — | 只降 ≈15% | — |

**账单只降 15%** 这点值得单独划线：每轮对话 AI 都要重读系统提示词 + 聊天记录，这部分装不装 skill 都得花。省下的八分之七省的是"**写**"，不是"读"。作者把这个最不性感的数字写在 README 首页——这比任何宣传语都可信。

视频模式省得更多：约 **18× fewer output tokens**、快 12×。因为一页图要写的结构信息，一段动画可以复用。

### 4. 程序排版顺带治好了一个老毛病

手写时 SVG 坐标是模型一个个算的，箭头指歪是常事。换成程序后：流程图用 **dagre** 布局算法排，时序图按标签文字实际宽度算间距——图起码整整齐齐。

更影响体验的是**自纠错**：草稿写错时 CLI 返回**行号 + 出错组件名 + 一段正确示例**，agent 一次就能改对，少了大量来回。

### 5. 它不乱出页面

官方给的触发样例：

| 你问什么 | 出来的页面 |
|---|---|
| 讲讲 TCP 三次握手 | 时序图 + 状态图 + 标志位表格 |
| 这个仓库的模块怎么组织 | 目录树 + 调用关系图 |
| Redis 和 Memcached 选哪个 | 对比表 + 结论 |
| 这段话哪里写得不好 | 逐句标注问题与改法 |
| Kubernetes 是怎么来的 | 时间线 |
| `ls` 怎么显示隐藏文件 | **不出页面**，一行问题一行回答 |

出不出由它自己判断：涉及多个概念 / 多个步骤 / 多个方案对比才出。想强制出，说一句"用 HTML 给我讲讲"即可。

## 技术细节

### 模型实际只写这么多

````markdown
---
title: TCP three-way handshake
---
## A Three-way handshake {span=2}
```sequence num
Client -> Server: SYN, seq=x
Server -> Client: SYN+ACK, seq=y, ack=x+1
Client -> Server: ACK, ack=y+1
note Client, Server: ESTABLISHED
```

## C State changes {span=2}
```flow LR
(CLOSED) -> LISTEN: passive open
LISTEN -> SYN_RCVD: get SYN / send SYN+ACK
SYN_RCVD -> *ESTABLISHED: get ACK
```
````

CLI 负责：选模板、排面板、套主题、`dagre` 排流程图、按标签宽度排时序图。

### 功能清单

- **代码即真实**：代码块可引用本机文件（`src=` `lines=`），页面上是真代码不是重抄；目录外文件和含密钥的文件会被拒绝
- **三种主题**：`blueprint`（工程图纸）、`shadcn`（干净卡片）、`paper`（长文阅读）；默认长文用 paper、图表用 blueprint；均有明暗模式，可自建或从主题目录取
- **单文件零依赖**：每页一个 `.html`，无 CDN、无 web font，离线可开、便于分享
- **多语言**：简繁中 / 英 / 日有独立按钮与字体
- **写作检查**：规则改自 **ASD-STE100**（航空维修手册的受控英语，限死词汇与句式让文档谁来读理解都一样），中英文都查——长句、啰嗦、被动语态、套话；默认只警告，严格模式才拦
- **页面上作答**：可对任意面板评论、在 `ask` 决策里选选项，Reply 按钮把你的评论合成一条消息贴回 agent
- **留源自证**：每页内嵌生成它的 Markdown，"Copy source" 可还原

### 常开模式（作者建议打开）

打开后，**每个结论都顺手带一页**：只要 AI 给出结论、总结、方案、对比、评审或解释，哪怕很短，也出一页 2–4 面板小页面，路径贴回复末尾，**不弹窗**。闲聊和无结论的一两句回复照旧。Claude Code 的 plan 模式不出页面。

默认关闭。打开方式是让 agent 把一条全局规则写进 rules 文件（`~/.claude/CLAUDE.md` 或 `AGENTS.md`）；关掉就删那条规则。

### 讲解视频支线

草稿里给每个画面配一句旁白，`am video` 生成播放器页面：图跟着旁白一步步画，镜头跟随 `[节点名]`，有章节条、倍速按钮。**配音塞在页面文件里，离线可播**。导出按钮在浏览器内编码下载（本地文件或 localhost，**不需要 ffmpeg**）；`--mp4` 要 ffmpeg，`--webm` 页面自编码不需要。只在你要时才做视频。

### 安装（四条路）

前提只有一个：**Node.js ≥ 20**。不用 `npm install`，CLI 已打包在 skill 里。

```bash
# 1) 让 agent 自己装（推荐）
# 把这句贴给 Claude Code / Codex / Cursor / OpenCode：
#   读 https://raw.githubusercontent.com/QingYunA/answer-me-with-html/main/INSTALL.md 并照做

# 2) 一行命令（支持 70+ agent，由 vercel-labs/skills 提供）
npx skills add QingYunA/answer-me-with-html
# 非交互：npx -y skills add QingYunA/answer-me-with-html -g -y -a <agent名>
# ⚠️ -a 参数别省——装错目录 agent 读不到

# 3) Claude Code 插件
/plugin marketplace add QingYunA/answer-me-with-html
/plugin install answer-me-with-html@answer-me-with-html

# 4) DeepSeek Harness 插件
dsh plugin --profile web add github:QingYunA/answer-me-with-html
```

**验证**：问「对比一下 Git 的 merge 和 rebase，分别什么场景用」应出对比表 + 结论；问「从输入网址到页面显示出来，中间发生了什么」应出完整流程图。两条都出页面就算通。

页面落在 `~/.answer-me-with-html/pages/`，一页一文件；多了用 `am clean` 清。设置全走斜杠命令（`am config`），无配置文件手改。更新是手动的，CLI 每天最多后台查一次 GitHub 版本，最多每三天提示一次，不上传任何用户信息。

## 关键洞察

### 作者自己的判断（原文结语）
> 它没让 AI 答得更对，内容还是那些，只是换了个看法。……真要挑毛病的话，它换的是看的角度，不是思考的深度。你自己的判断力还是得留着。

这句是这个项目最诚实的地方。**它是更好的"看法"，不是更好的"答案"。** 解释、对比、流程类问题看图确实省劲；想把回答发给别人，发一个 HTML 文件比截一长串聊天记录像样。

### 我的评估（针对本地环境）

| 维度 | 判断 |
|---|---|
| 真实热度 | 4,435⭐ / 24 天 / 日更，热度真实；但"神级"是公众号标题话术 |
| 技术含量 | 中高。真正的价值不是"生成 HTML"，而是**把"内容/排版"职责切开**这个架构决定 + 自纠错回路 |
| 最大的坑 | **输出与输入隔离**。它只影响回答的呈现层，对知识库、代码、决策质量零增益 |
| 与本地栈的关系 | Node v22.23.3 已满足；`~/.answer-me-with-html` 为空，未装；Hermes 不在此 skill 声明的宿主列表内（Claude Code / Codex / Cursor / OpenCode / Pi / DeepSeek Harness） |
| 最贴合小谭的用法 | 不是日常问答，而是**归档产物升级**——把快报/汇报稿从 Markdown 表格变成一页可发的 HTML 单文件 |

### 与知识库既有主题的接缝

这是本周第三条"生成 → 呈现"链路的文章，三条正好是三层的不同切面：

| 层 | 工具 | 解决 |
|---|---|---|
| 生成 | Step Code / WorkBuddy | 一句话造出工作台 |
| 分发 | StepPage / 花生壳 Drop | 让手机/别人能打开 |
| **呈现** | **answer-me-with-html** | **让长内容变得能读** |

它和 `baoyu-infographic`、`claude-design`、`architecture-diagram` 有重叠，但定位不同：那三个是"我要做一张图/一个设计"，这个是"**AI 回答时自动带图**"——触发时机在对话流里，不在任务开始时。

## 参考

- 仓库：https://github.com/QingYunA/answer-me-with-html
- 官网 / 主题目录：https://answer-me-with-html.com/
- 中文 README：https://github.com/QingYunA/answer-me-with-html/blob/main/README.zh-CN.md
- 参考文档：https://github.com/QingYunA/answer-me-with-html/blob/main/docs/reference.md
- Karpathy 原帖：https://x.com/karpathy/status/2105819303471976479

---

原文封面：

![[assets/2026-10-25-answer-me-with-html-AI回答变一页HTML/cover.jpg]]
