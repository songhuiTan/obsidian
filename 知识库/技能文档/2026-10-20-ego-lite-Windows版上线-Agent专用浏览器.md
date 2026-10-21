---
title: "评论区催爆的 ego lite Windows 版终于来了！"
source: "程序员阿江-Relakkes（抖音）"
source_url: "https://www.douyin.com/video/7698349679792164136"
short_url: "https://v.douyin.com/fhWQCe7BJEI/"
date: "2026-10-20"
duration: "07:40"
series: "AI人工智能 第76集"
tags: [AI浏览器, egolite, Agent, 浏览器自动化, 登录态, Hermes集成, 抖音归档]
---

# 评论区催爆的 ego lite Windows 版终于来了

> **作者**：程序员阿江-Relakkes（粉丝 3.1万 / 获赞 13.5万）
> **发布**：2026-10-20 12:00 · 时长 07:40 · 合集「AI人工智能」第 76 集
> **数据**：👍 245 · 💬 16 · ⭐ 188 · ↗ 42（归档时）
> **原视频**：https://www.douyin.com/video/7698349679792164136

## 一句话

`ego lite`（一个专为 AI Agent 设计的 Chromium 浏览器，Win/Mac 双端）终于在 Windows 上线。作者把它设成**默认浏览器**替换 Chrome，实测在 Windows 上开两个 Space 并行跑真实任务。**免费**（评论区作者亲答）。

## 文案全文

之前 Codex 省 skill 那期视频评论区一直有人问："Windows 什么时候能用？"这次终于来了！

这段时间我一直在高强度使用 ego lite，已经把它设成默认浏览器，替换了原来的 Chrome。平时自己上网、查资料，需要处理重复工作时，就让 Agent 在独立的 Space 里操作网页、复用登录态，前台继续干自己的事。

这期直接在 Windows 上跑两个真实任务：一边整理我自媒体账号最近 30 天的视频链接、播放、点赞和评论数据，做自媒体复盘；另一边测试我自己的开源项目 **cc-haha**，让 Agent 操作本地网页，完成前端 UI 自动化测试。两个 Space 同时跑，最后一起看结果。

视频里还演示了 Windows 版安装、浏览器数据迁移、ego-browser Skill 接入和垂直标签页，并回应上期评论区的问题：Codex、Chrome 也能操作网页，我为什么还是把 Ego Lite 留下来当主力？结合自己的使用经历，聊聊**登录态与网站验证、多 Space 并行、人机协同与中途接管，以及执行速度和 Token 消耗**。

一直在等 Windows 版的朋友，可以拿自己的真实任务试一试。

`#AI新星计划 #egolite #egolitewindows版本 #效率工具 #AI工具`

## 章节要点（官方章节条）

| 时间 | 内容 |
|---|---|
| 00:00 | Windows 版终于来了 |
| 01:05 | 主力浏览器体验 |
| 01:47 | 安装与数据迁移 |
| 02:32 | 双任务并行实战 |
| 04:08 | 实战结果与原理 |
| 04:56 | 四大核心优势 |
| 06:59 | 长期使用总结 |

## 两个实测任务（视频主体）

**任务 A · 自媒体复盘**：整理自己账号最近 30 天的视频链接、播放、点赞、评论数据。
→ 典型"要登录态但又没 API"的场景，正是需要真实浏览器会话的地方。

**任务 B · 前端 UI 自动化测试**：测试作者自己的开源项目 `cc-haha`，让 Agent 操作**本地网页**跑 UI 测试。
→ 说明 ego lite 不只管公网站点，`localhost` 开发态也能接管。

两个 Space 同时跑，最后汇总看结果——这是"并行"而非"更快"的价值主张。

## 评论区实况（含作者回复）

- **@LASTDANCE**："继承用户登录态和 space 这两个能力 playwright 不都已经实现了么"——这条质疑最值得留意（见下方战略分析）
- **@iAi**："全交给 agent 还是经常会给你搞出来一些惊喜"（作者赞过）
- **@floria**："ego 容易让我 mac 键盘失灵，重启才行"——⚠️ Mac 端有键鼠输入异常反馈
- **@久**："感觉有时候 ego 操作还是很慢"；另推荐了 `lightpanda`（同类 AI 浏览器），作者回复"有机会看看"
- **@哈鼎**："这个浏览器收费？" → **作者回复：免费**
- **@大头C**："这个真的很好用"（作者赞过）

## 官方产品事实（我另行核实，非视频内容）

来源：`lite.ego.app` 官方文档 + 官网

- **形态**：Chromium 内核浏览器，装着就是你的日常浏览器，扩展／历史／登录态全部继承。生态：Discord 社群、GitHub `citrolabs/ego-lite`。
- **下载**：macOS 正式版，**Windows 为 beta**。
- **核心机制**
  - **Space**：每个任务一个隔离浏览上下文。Agent 在自己的 Space 干活，不抢你的标签页，不弹窗劫持前台。
  - **Snapshot**：Agent 每次动作后重新读取页面快照再继续（避免旧引用失效）。
  - **ego-browser**：给 Agent 的 runtime，第一版启动时自动扫描本机已装 Agent，把 skill 写进 `~/.agents/skills` 与 `~/.claude/skills/`。思维模型是 JS helper：
    ```js
    await useOrCreateTaskSpace('check a page')
    await openOrReuseTab('https://example.com', { wait: true })
    cliLog(await snapshotText())
    ```
- **定位边界（官方明说）**：登录、验证码、支付、最终提交由**人**来做，Agent 到付款页就停。

## 🔥 与我们直接相关：官方有 Hermes Agent 专章

`https://lite.ego.app/document/en/docs/windows/hermes-agent` —— ego lite 官方文档里**专门写了 Hermes Agent 的接入教程**（Windows）：

1. Settings → Safety → Approvals，Approval Mode 设为 **Smart**
2. 输入框打 `/ego-browser`，**后面留一个空格**再写任务（不然 Hermes 报"未知命令"）
3. Agent 在 ego lite 的 Space 里开 Airbnb 搜东京房源，返回带来源链接的三条对比表
4. 需要登录/验证码时，人进 Space 处理，再让它继续

排错条目也点名了 Hermes：skill 不出现（重启 Hermes）、打不开 ego lite（查 Approvals 权限）、Airbnb 要登录（手动进 Space 完成）。

**当前本机状态**：未安装 ego lite（`/Applications` 无、`~/.agents/skills` 只有 `web-access`）。官方提供 macOS 版，若要试跑可直装。

## 战略分析

### 1. 它解决的到底是不是新问题？

`@LASTDANCE` 那句质疑成立一半：Playwright 确实有 `storageState`（持久化登录态）和多 `context`（隔离）。但两者的重心完全不同——

- **Playwright**：面向测试的**无头自动化框架**，登录态要你手工导出/刷新，隔离是 context 不是"你的日常浏览器"。
- **ego lite**：面向"替人干活的 Agent"的**有人浏览器**，登录态是你真实在用的 cookie/Profile（含扩展、验证码后状态），隔离是可视化可接管的 Space。

差别不在能力清单，在**"人机共存"**：Playwright 跑起来你插不上手，ego lite 的设计前提是"你随时可以进去接管"。这才是它敢拿"替代 Chrome"当卖点的原因，也才是评论里"操作慢"这个缺点的来源——慢是慢在真实渲染 + 人对齐成本，不是慢在没优化。

### 2. 对我们这套体系的价值排序

| 优先级 | 用途 | 理由 |
|---|---|---|
| **高** | 补上"必须登录才看得见"的数据源 | 小红书/公众号后台/知乎/微博 这些现在全是 Cookie 硬塞，靠 r.jina.ai 和 playwright 半残，ego lite 的登录态继承是正解 |
| **中高** | Hermes 内跑本地 Web UI 回归 | 现在验证 HTML 产物（deck、dashboard）靠截图＋肉眼；用 Space + Snapshot 可让 Agent 自己点一遍 |
| **中** | 替代/补强 `blocked-page-recovery` | 对付 403/bot wall 时，"真人浏览器已登录"比换 UA 有效得多 |
| **低** | 替人做后端操作（CRM、报表刷新） | 我们目前没有这类 SaaS 日常，用不上 |

### 3. 风险与坑（不是官方宣传口径）

- **Mac 端键鼠失灵**（评论区 @floria 实测）——这是**输入层**故障，比慢更严重，装前得有心理准备。
- **Windows 还是 beta**。视频讲的是 beta 版能力，稳定性别当正式版看。
- **`ego-browser` skill 会被写进 `~/.agents/skills` 和 `~/.claude/skills/`**——这是把你本机 Agent 自动化的一把钥匙交给了软件自动安装。结合上次 XYskill 里看到的 `skill-audit`，这类"自动注入 skill"正是该被安检的东西：装完先看它写进去的 SKILL.md 到底允许 Agent 做什么。
- **登录态继承 = 高权限**。你的所有登录会话在一个"Agent 可以驱动"的浏览器里，等于把钓鱼面/误操作面放大。官方也承认必须保留人工确认环节，别图省事关掉。
- **Token 成本**：Snapshot 全文喂给模型，长页面很贵。作者第四大优势声称省 Token，但这是相对"截屏视觉"方案而言；文本快照在长列表页未必便宜。

### 4. 建议动作（我做的话）

1. **先在 Mac 装，用一次性小号登录一个站点试跑**，确认三件事：键鼠是否正常、Space 是否真不抢前台、Snapshot 在长页面的 token 开销。
2. **装完立刻审 `~/.agents/skills/ego-browser/SKILL.md`**，跟现有 `web-access` skill 比对，看权限声明是否越界。
3. 若通过，接入点选**最痛的那个数据源**（后台数据类），别一上来就铺全局。
4. **不替换默认浏览器**。作者能替换是个人习惯；我们这类环境把 Agent 浏览器和日常浏览器混在一体，风险收益不划算——用它的 Space 能力，但日常浏览继续隔离。

### 5. 一句话结论

**Win 版上线本身不算新闻，新闻是"Agent 专用浏览器已经开始抢默认浏览器这个位置"**。对我们是好消息：登录态类数据源终于有正解了。但要按"高权限软件"对待——先审 skill、小号试、别换默认浏览器。

---

## 归档日志

- 2026-10-21 归档。抓取方式：抖音短链 → 浏览器渲染取文案/章节/评论区 + 官方文档核实产品事实。
- 视频主页：`https://v.douyin.com/fhWQCe7BJEI/` → `https://www.douyin.com/video/7698349679792164136`
- 官方文档（Hermes 专章）：`https://lite.ego.app/document/en/docs/windows/hermes-agent`
- 相关：[[技能文档/2026-10-11-两个亿的操盘手把自己蒸馏成48个Skill-XYskill]]（`xy-skill-audit` 安检思路）
