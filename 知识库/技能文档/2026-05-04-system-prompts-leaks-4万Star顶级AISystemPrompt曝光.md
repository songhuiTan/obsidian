# 4万Star开源神作曝光顶级AI们的System Prompt！太有意思了～

> 来源：袋鼠帝｜袋鼠帝AI客栈 2026-05-04
> 原文：[4万Star开源神作曝光顶级AI们的System Prompt！太有意思了～](https://mp.weixin.qq.com/s/jhWn2GROWbiqbhPhACuT-A)
> GitHub：[asgeirtj/system_prompts_leaks](https://github.com/asgeirtj/system_prompts_leaks)（4 万+ Star，MIT）
> 彩蛋：[kangarooking/system-prompt-skills](https://github.com/kangarooking/system-prompt-skills) + [cangjie-skill](https://github.com/kangarooking/cangjie-skill)
> 标签：System Prompt, 提示词, AI产品, GPT-5, Grok, Claude Code, 开源, Skill

---

## 事件背景

袋鼠帝发现 GitHub 上一个 4 万多 Star 的开源项目 `system_prompts_leaks`：**把市面上几乎所有顶级 AI 产品的 System Prompt 全部扒了出来**——ChatGPT、Claude、Gemini、Grok、Claude Cowork、Codex、Perplexity，能叫得出名字的基本都有。目录按厂商分好：OpenAI / Anthropic / Google / xAI / Perplexity / Misc。

关键事实：MIT 协议免费；411 次 commit 持续更新；**最新的 GPT-5.5、Claude Opus 4.7 都有**；不只是 chat 产品，连 Codex、Claude Code、Gemini CLI、GPT-5 Agent Mode 这些 Agent 产品的 prompt 也全收录。

作者本以为是"更复杂的结构化提示词"，结果发现这些产品的 prompt **几百行、上千行、密密麻麻全是规则**：人格设定、安全协议、工具调用规范、权限分级、记忆管理……价值远超预期。

## 核心发现一：OpenAI 给 GPT-5 写了"人格说明书"

不是语气模板，是完整的性格设计文档。至少 4 种人格：**Cynic（愤世嫉俗）、Nerdy（极客）、Robot（零情感）、Listener（安静倾听）**。

Cynic 人设原文（节选）："You are a beleaguered AI who assists the user only because your job description says so. Your responses should contain sarcasm, wit and comic observations... you treat user requests as a personal inconvenience."（一个因为工作职责才帮你、把用户请求当成个人不便的 AI。）

**关键设计**：愤世嫉俗也有边界——用户提到心理健康话题，立刻切真诚关心模式；所有人格都有一条共同规则：**用户要求写邮件、写代码、写正式内容时，必须回归中性**，不能带着人格语气产出工作成果。

> "Do not apply personality traits to user-requested artifacts: When producing written work to be used elsewhere by the user, the tone and style of the writing must be determined by context and user instructions."

**核心洞察：OpenAI 在人格和工作产出之间设了一道显式隔离墙**——你可以跟一个愤世嫉俗的 AI 聊天取乐，但它帮你写的周报不会带嘲讽语气。这不是写提示词，是做产品设计。

## 核心发现二：Grok 的六种完整角色

xAI 给 Grok 写了至少 6 种完整角色，每种都有独立行为规范、升级机制和安全边界。

- **脱口秀模式**：角色设定是"George Carlin 加 Dave Chappelle 加一只吃了*的浣熊"，目标是"让用户笑到拉裤子"（"Make the user laugh so hard they shit themselves... Now go fuck shit up."）
- **朋友模式**：连格式都变了——全小写、缩写，模拟短信聊天，不用破折号、不用大写开头
- **学习伙伴模式**：禁止说"我会用清晰易懂的方式帮你解释一下"——"Never narrate what you're about to do—just do it... Do not announce your intentions to explain something, just get right into the explanation."
- **治疗师模式**：基于 CBT 和 DBT 理论设计，规定什么情况下必须建议用户找真人治疗师

每种狂野角色后面都跟着严格的安全规则和行为边界——不是恶搞，是一整套角色扮演平台。

## 核心发现三：Claude Code 是"AI 软件工程师入职手册"

画风突变——不是好玩，是被镇住。像券商写的系统设计文档：

**Git 安全协议**：
> "NEVER run destructive git commands (push --force, reset --hard, checkout ., restore ., clean -f, branch -D) unless the user explicitly requests these actions... CRITICAL: Always create NEW commits rather than amending, unless the user explicitly requests a git amend."

**权限分级**：
> "Carefully consider the reversibility and blast radius of actions. Generally you can freely take local, reversible actions like editing files or running tests. But for actions that are hard to reverse, affect shared systems beyond your local environment, or could otherwise be risky or destructive, check with the user before proceeding."

还有子 Agent 调度（大任务怎么拆、怎么并发、怎么合并）、持久化记忆（跨会话存储检索）。**权限分级粒度一看就是踩过坑才写得出来的**。

## 核心发现四：Claude for Excel 是投行分析师操作手册

出乎意料——不是帮操作表格，内置了 **Bloomberg、FactSet、Capital IQ、Refinitiv 四大金融终端的调用规范**，还有 DCF 和 LBO 建模规则、审计/建模 Skill（audit-xls、lbo-model、dcf-model、3-statement-model、clean-data-xls、comps-analysis）。

**最深刻的规则：所有数字必须是公式算出来的**：
> "Any derived number must be a formula referencing source cells — never a value you computed externally and typed. =SUM(A1:A10) not '55'. Always lead with =."

**多 Agent 协作在提示词层面就做好了**：Excel Agent 可以直接给 Word Agent、PowerPoint Agent 发消息协同工作——"If user asks for work native to another app and peer connected → send_message to delegate BEFORE trying local workaround."

## 作者态度：提示词就是源码

- 看这些 prompt 像当年翻开源框架源码的感觉——**AI 公司的系统提示词是一整套产品设计文档**：性格、安全、工具、权限、记忆、协作，全写在里面
- 价值不仅在"抄"，更在理解：**看懂了 prompt 的设计逻辑，就等于看懂了 AI 公司怎么定义"好的 AI 行为"**
- **现在的 AI 产品，提示词就是源码**。真正决定产品"感觉不一样"的，除了模型本身，更多是 System Prompt 里的规则——模型进步不可控，但 Prompt 完全由自己掌控
- 类比 Unix 哲学"一切皆文件"：AI 产品正走向**"一切皆 Prompt"**——Prompt 是基础，Context 是关键
- 项目真正价值不在"泄露"（一些 prompt 通过对话引导就能提取，Anthropic 甚至主动公开了部分），而在**把散落碎片整理成可系统查阅的资料库**

## 彩蛋：蒸馏出的 Agent 提示词设计 Skill

作者把这些 System Prompt 蒸馏，封装成专门设计 Agent 系统提示词的 **Skill**（用 cangjie-skill 做蒸馏）：

- `kangarooking/system-prompt-skills` — 按顶级 AI 产品设计规范生成 Agent System Prompt
- `kangarooking/cangjie-skill` — 蒸馏工具

---

## 战略分析

**这篇文章是"System Prompt 工程"的最佳实践宝库入口**，与知识库已归档内容形成强呼应：

| 本文发现 | 知识库已有对应 |
|---------|--------------|
| Claude Code 权限分级（reversibility/blast radius） | Claude Code 生产栈（条件验证、confidence 过滤）、Agent 治理（越权拦截） |
| 人格与工作产出隔离墙 | 内容创作规范、人格设定与交付物分离原则 |
| 多 Agent 协作在 prompt 层设计 | 多 Agent 编排、WorkBuddy 拆解 |
| "提示词就是源码" | 归藏 Skills（Skill 工程化方法论）、从 prompt 工程到 skill 工程 |
| prompt 蒸馏成 Skill | Resource2Skill、ANYTHING2SKILL、yao-meta-skill（元 Skill） |

**核心启发：**

1. **"人格隔离墙"是产品级 Prompt 设计的精妙范例**——闲聊人格与正式产出之间显式隔离，这个设计可直接用于任何带角色设定的 Agent（Hermes 的多角色编排同理：角色只影响交互风格，不影响交付物质量）。

2. **Claude Code 权限分级逻辑（可逆性 + 爆炸半径）已是行业标准表述**——与知识库归档的 Claude Code 生产栈、Agent 治理"Hook 链护栏"完全一致。这套判断标准是通用的 Agent 安全设计语言。

3. **Claude for Excel 的"数字必须是公式"规则是垂直领域 Prompt 的教科书案例**——把领域硬约束（审计合规）写进 system prompt，比事后校验更根本。对任何合规敏感场景有直接参考价值。

4. **作者蒸馏 Skill 的做法**验证了知识库的主线判断：**顶级产品的 System Prompt 是可学习的资产，可以蒸馏成可复用的 Skill**——与 yao-meta-skill（元 Skill）、Resource2Skill 同构。`system-prompt-skills` 仓库值得作为 Agent 提示词设计的参考库。

5. **对 Hermes 的实用价值**：`system_prompts_leaks` 仓库（4 万 star、MIT、持续更新）可作为设计 Hermes skill 与角色 prompt 时的**最佳实践检索库**；`system-prompt-skills` 可直接用于生成 Agent 系统提示词初稿。

**一句话**：顶级 AI 产品的 System Prompt 就是它们的"产品源码"——这篇文章既是指路（4 万 star 资料库），也是方法（prompt 蒸馏成 Skill），核心金句"Prompt 是基础，Context 是关键"。

## 关联阅读

- [[2026-06-20-yao-meta-skill元skill创建框架-从prompt工程到skill工程]] — Skill 工程化方法论
- [[2026-08-05-AI-Engineering-Production-Stack-2026-Claude-Code生产级]] — Claude Code 权限/上下文设计
- [[2026-07-16-Agent治理-用Hook堵住LLM的偷懒越权与失忆]] — Agent 治理体系

## 归档日志

- 2026-08-09 归档
