---
type: concept
title: 狂揽 11 万 Star！给 Codex 装上这个神级 Skill，我终于把网页改出了自己的风格！
date: '2026-11-14'
source: 顾北 AI
source_url: 'https://mp.weixin.qq.com/s/ODgUcOctQlh1iNRD8Oqqqg'
ingested_via: put_page
ingested_at: '2026-11-18T03:29:19.245Z'
source_kind: put_page
tags:
  - 前端
  - 设计审查
  - Agent Skill
  - Codex
  - Impeccable
  - UI设计
---

# 狂揽 11 万 Star！给 Codex 装上这个神级 Skill，我终于把网页改出了自己的风格！

> 作者：顾北（公众号「顾北 AI」）｜发布于 2026-11-14 11:58

## 概述

作者在折腾自己的「内容创作平台」（文章数据、粉丝趋势、各类图表），功能全部做完，但页面怎么看都有一股"AI 模板味"——熟悉的卡片、表格、侧边栏组合，不像自己的工作台。让 Codex 改过几轮、也试过别的 UI Skill，效果都不满意。

转折点是把 **Impeccable** 这个 UI/UX 设计 Skill 装进了 Codex，并且**换了一种工作方式**：不再丢一句"帮我设计得高级一点"，而是先确认设计方向（`init` → `shape`），再针对具体不满意的区域逐个处理（`adapt` / `critique` / `live` / `generate`）。

改造结果：靛蓝紫配色 + 山川背景 + 自己的卡通 IP 头像，指标布局统一，终于做出个人品牌辨识度。

![改造前](../assets/2026-11-14-impeccable/img_001.jpg)

![改造后](../assets/2026-11-14-impeccable/img_002.jpg)

## Impeccable 是什么

一套开源的 **UI/UX 设计 Skill**，官方定位："The design vocabulary for your coding agent"——给编码 Agent 一套能听懂的设计词汇。

核心能力三块：

1. **用词精确地"指挥"** —— 不是"让它更 pop"，而是 `polish`（收尾精修）、`bolder`（放大安全设计）、`quieter`（收敛过于吵闹的设计）、`distill`（删掉不需要的）、`typeset`（修字体层级）、`layout`（修间距节奏）等 24 个明确动词。
2. **审查员** —— `critique` 按可用性启发式打分并排优先级；`audit` 检查无障碍、性能、主题化、响应式，按 **P0（阻塞）→ P3（打磨）** 分级。
3. **AI 味探测器（slop detector）** —— **62 条确定性规则**（无需 LLM、无需 API Key），识别那些跨模型反复出现的模板套路：全站 Inter 字体、紫蓝渐变、卡片套卡片、彩色背景上的灰字、每个标题上一个圆角方块图标。

> 项目地址：https://github.com/pbakaus/impeccable
> 官方文档：https://impeccable.style/docs

**仓库事实核查（2026-11-18 实测 GitHub API）**

| 项目 | 数值 |
|------|------|
| Stars | **112,296**（文章说"11 万左右"，属实） |
| Forks | 6,696 |
| Watchers | 218 |
| Open Issues | 52 |
| License | **Apache-2.0**（标准 OSI 开源，非 source-available） |
| 主语言 | Rust（CLI 本体） |
| 建仓时间 | **2025-11-16**（比本文早整一年，是真实产品持续开放，不是临时炒作发布） |
| 最近推送 | 2026-11-18（当日仍在活跃开发） |
| Topics | agent-skills / anti-slop / design / ai / slop |

作者 pbakaus = Paul Bakaus（jQuery UI 原核心开发者、Google/Chrome DevTools 团队出身），项目有真实的工程背景，不是无名小号。

## 安装 Impeccable

官方两种装法，作者选了**项目级手动安装**（装到已有项目里，方便后续管理）：

- 官方推荐：`npx impeccable install`（项目根目录执行），重载工具后运行 `/impeccable init`（Codex 上是 `$impeccable init`）
- 作者用的手动方式：把仓库的 `.agents/skills/impeccable` 完整复制到当前项目的同路径下

作者给 Codex 的实际提示词：

```
请按官方手动复制方式安装 Impeccable：
https://github.com/pbakaus/impeccable

将仓库 .agents/skills/impeccable 完整复制到当前项目同路径。
已有同名目录先保留，只装项目级，不修改页面。
完成后读取 SKILL.md，确认是否能正常调用。
```

装完重开会话，输入 `$impeccable` 验证能识别。

> **作者强烈建议**：装之前先用 Git 保存当前代码，UI 调整不满意可随时回退。

**官方支持的工具清单**（README 实测）：Claude Code、Cursor、Codex CLI、GitHub Copilot、Devin、Replit Agent、Gemini CLI、Grok Build、OpenCode、Google Antigravity、Kiro、Trae、Rovo Dev、Qoder、Mistral Vibe、Pi、**Hermes Agent**、DeepSeek Harness、Veto。

## 动工前：先把设计方向聊清楚

作者最重要的反思是：**"我自己也说不清喜欢的样子到底是什么"**。所以第一步不是改代码，而是让 Agent 先理解项目：

```
$impeccable init
```

`init` 会读代码，只追问它推断不出来的东西（产品给谁用、要完成什么、有什么约束），然后写出 `PRODUCT.md`。作者补充了「文章分析、粉丝分析」等场景，以及**必须保留真实数据和现有交互**的硬约束。

第二步 `shape` 只讨论、不动代码：

```
$impeccable shape

请读取 PRODUCT.md 和现有页面，规划 UI 重构。
希望页面专业、克制，有自己的品牌辨识度。
保留现有功能和真实数据，给出设计方案与取舍。
这一步只讨论，不修改代码。
```

作者另外借助 ChatGPT 做了桌面端和移动端的视觉参考，最终定下 **靛蓝紫 + 山川背景 + 个人 IP** 三件套，再让 Codex 动手重构首页。

![设计方向与页面](../assets/2026-11-14-impeccable/img_003.jpg)

## 页面不好看却说不出原因？先让它挑毛病

第一版做出来后，作者换到另一块显示器发现布局没适配窗口宽度——用 `adapt` 而不是含糊的"帮我修一下"：

```
$impeccable adapt

请检查首页在桌面、平板、手机下的布局。
修复侧栏挤压、卡片重排及内容溢出。
保留真实数据、筛选、跳转和当前设计风格。
完成后在不同窗口宽度下实际检查。
```

布局改善后，仍有卡片"看着别扭但说不清哪里别扭"，这时候用 `critique`：

```
$impeccable critique

请审查当前首页的卡片、间距、字号、图表和文章列表。
指出具体问题、原因及修改建议，按优先级排序。
先给报告，不要修改代码。
```

它把模糊的直觉拆成了具体问题。作者的关键做法：**先要报告、再挑最重要的改**，至少知道这一轮在解决什么，而不是盲目刷新页面。

![critique 报告：按 P1/P2 分级的双 Agent 设计评审](../assets/2026-11-14-impeccable/img_004.jpg)

## 最好玩的部分：Live 直接在浏览器里点着改

整体方向确定后，先用 `$impeccable document` 把设计规范整理进 `DESIGN.md`，再启动本地页面：

```
$impeccable live
```

浏览器会浮出一条黑色工具栏。操作流程：

1. 点击 **Pick**，再点想调整的卡片
2. **看清高亮范围**，别只选中一个标题或内部 div
3. 在 **Freeform** 菜单选模式，输入想怎么改
4. 保持 **×3**，点黄色箭头，对比三个方案
5. 满意点 **Accept** 写回代码，不满意放弃

![Live 工具栏与选中元素](../assets/2026-11-14-impeccable/img_005.jpg)

最常用的五个模式：

| 模式 | 用途 |
|------|------|
| **Layout** | 布局、分组、留白不合理 |
| **Typeset** | 标题、数字、说明文字不好读 |
| **Polish** | 布局已定，只精修细节 |
| **Adapt** | 大屏/小屏响应式问题 |
| **Freeform** | 不确定选什么，直接描述困扰 |

例如选中一张指标卡、选 Polish：

```
这张卡片的信息层级不够清楚。
请优化数字、标题和辅助文字的排版、间距。
保留当前配色、真实数据和功能，只修改选中区域。
```

三个版本摆在一起看，比在对话框里反复说"再高级一点"直观得多。

![方案对比与整页效果](../assets/2026-11-14-impeccable/img_007.jpg)

## Pick 选不中整行时的替代方案：generate

作者想把首页五个核心指标统一——原来四个指标挤一组、「累计分享」单独成卡，观感割裂。但 Live 的 Pick 总是选中内部元素，选不到整个指标区。

![Pick 选不中整行：只高亮到内部 form 元素](../assets/2026-11-14-impeccable/img_006.jpg)

于是他退出 Live，回到 Codex 用 `generate` 直接描述整块区域：

```
$impeccable generate 3 layout variants of the account overview KPI section

请定位首页五个核心指标的共同父容器。
探索等宽卡片、间距和响应式重排。
保留真实数据与交互，不修改其他区域。
```

这次成功定位了整行，最后选了五张等宽卡片。

![generate 执行回报：三套指标区方案与多宽度响应式验证](../assets/2026-11-14-impeccable/img_008.jpg)

**失败也很正常**：后来让 `generate` 重排文章表格，三个版本他都没选——有的标题被挤成好几行，有的把桌面表格拆成卡片导致无法横向比较。

> 作者的关键结论：**当三个版本都不好看，就别硬选。** 下一轮应该明确告诉它：「桌面端保留表格、标题列加宽、指标右对齐」，问题才会真正收敛。

![失败案例：表格重排三版均不可用](../assets/2026-11-14-impeccable/img_009.jpg)

## 写在最后

作者最珍视的不是某个"神奇命令"，而是**找到了一种改页面的方式**：

> 先确定自己想要什么，再让 AI 诊断问题、给出几个方案，最后自己来选。

页面慢慢有了自己的风格，但**好不好用还得在实际浏览器里检查**。给新手的建议：从一张卡片开始试，不要一上来就重做整个网站。

## 战略分析

**这篇文章的真正价值不在工具，而在工作流。** Impeccable 的 24 个命令本质是把"设计沟通"从形容词变成了动词——`polish` 而不是"高级一点"，`distill` 而不是"简洁一点"。作者把它拆成了三段式：**方向对齐（init/shape）→ 诊断分级（critique/audit）→ 局部方案对比（live/generate）**。这套结构和我们已有的 doc/PPT 类工作流高度同构：先建受众模型，再产内容，再验证。差别在于 Impeccable 把"验证"做成了可视化的方案对比，人只需要选，不需要描述。

**成熟度判断：可以放心用，不是炒作项目。** 112K stars / Apache-2.0 / 建仓于 2025-11-16（比文章早整整一年）/ 当日仍在推送，作者 Paul Bakaus 是 jQuery UI 原核心 + Chrome DevTools 团队出身。特别是它的 62 条 slop 检测规则**是确定性的、不调 LLM、不需要 API Key**——这意味着它能作为 CI 里的一个静态检查跑，而不只是一个"对话框技巧"，这是它和绝大多数 UI prompt 库的本质区别。

**与我们现有栈的整合点（重点）**

| 维度 | 现状 | Impeccable 能补什么 |
|------|------|---------------------|
| **Agent 兼容性** | Hermes Agent | README 明确列出 **Hermes Agent** 为支持工具——可直接接入，不需要适配层 |
| **产出质量闸门** | 归档/PPT/HTML 交付物靠人工目检 | `audit` 按 P0→P3 分级 + 62 条确定性规则可做"交付前自动体检" |
| **设计一致性** | 无 DESIGN.md 类规范沉淀 | `document` 生成 DESIGN.md，`extract` 把重复模式抽进设计系统——正好补上"格式规范执行"这一环 |
| **浏览器闭环** | browser_exec 可截图/操作 | Live 模式的"点选→三方案→Accept 写回"是纯文本对话做不到的反馈回路 |
| **风险点** | — | Live 模式需要本地 dev server（Vite/Next/SvelteKit/Astro/Nuxt/纯 HTML）；Accept 会写源码，必须先 git commit |

**差距与借鉴点：** 我们现在做 HTML/前端交付物（claude-design、sketch、brain-pdf 这类）时，质量完全依赖我自己的审美判断，没有外部可复现的评分标准。Impeccable 的 `critique` 报告（文章里那张 P1-01/P1-02/P2-03 分级表就是实例）给出了一个可归档、可对比的中间产物——这比"我改好了你看看"要可靠得多。**建议的落地方式：不是装成 Codex skill，而是走 `npx impeccable install` 装进 Hermes Agent，然后在需要出 HTML/前端产出的任务里，把 `critique` 报告作为交付物的一部分一起给出来。**

## 关联阅读

- [[2026-11-12-Modern-Web-Guidance-给AI装现代Web知识大脑]] — 同一条路线：给 Agent 补"现代 Web/设计的领域知识"，那是知识层，这篇是执行层
- [[2026-06-26-Penpot-开源设计平台-AI直接读设计稿]] — 设计稿到代码的另一条路（结构化读取 vs 语义化指挥）
- [[2026-06-16-9张AI生成的图吃透任何一个前端项目]] — 前端项目理解，与本文的"改了再说"形成前后端分工

## 归档日志

- 2026-11-18 归档｜来源：微信文章（浏览器 `#js_content` 提取，16 张配图全量下载；Firecrawl 首调只返回标题，改用 browser 路径）
- 仓库事实已通过 GitHub API 独立核实（stars/forks/license/created_at/pushed_at），非文章口径
- 文章发布日期 2026-11-14 11:58 取自页面 `#publish_time`
