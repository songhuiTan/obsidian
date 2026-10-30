---
title: "谷歌开源神器：轻松完成企业级 Agent 从搭建到部署全流程！"
source: "Java基基"
author: "X"
source_url: "https://mp.weixin.qq.com/s/P5jBCwbxDoibeF_23tw9YQ"
repo: "https://github.com/google/agents-cli"
docs: "https://google.github.io/agents-cli/"
date: 2026-10-30
tags: [agents-cli, google-cloud, adk, agent-skills, skill, eval, deploy, gemini, 归档]
---

# 谷歌 agents-cli：把云平台部署规程塞进 coding agent 的上下文

![封面](assets/2026-10-30-agents-cli/img_001.jpg)

> **一句话定位**：`agents-cli` 不是 coding agent，是**给 coding agent 用的工具**——把「在这个平台上怎么搭、怎么评、怎么上线」写成规程，塞进 Claude Code / Codex / Gemini CLI 的上下文里，让它按规程办事而不是凭印象发挥。
> 原文：Java基基《谷歌开源神器：轻松完成企业级 Agent 从搭建到部署全流程！》(2026-10-30 11:55)

---

## 0. 归档日实测仓库状态（2026-10-30，GitHub API 口径）

| 指标 | 值 |
|---|---|
| 仓库 | [`google/agents-cli`](https://github.com/google/agents-cli) |
| Star / Fork | **6,227 ⭐ / 716 fork** |
| License | Apache-2.0 |
| 语言 | Python（PyPI 包名 `google-agents-cli`） |
| 建库 | 2026-04-08 |
| 最近推送 | 2026-10-27 |
| 最新版本 | **v1.12.0**（2026-10-27），前序 v1.11.0（10-20）、v1.10.0（10-12）、v1.9.0（10-06）、v1.8.0（09-30） |
| 官方文档 | https://google.github.io/agents-cli/ |

> ⚠️ 文章写的是「v1.11.0 / 六千一百多 Star」，归档时**已经是 v1.12.0 / 6,227⭐**，且十月内连发三版（10-06、10-12、10-20、10-27）——版本迭代速度是判断这类工具成熟度的第一个指标。

---

## 1. 它解决的真问题：模型没读过你们这一个项目的部署手册

文章开头的场景描述是全文最值钱的一段：

> 你让 Gemini CLI 或者 Claude Code 把 Agent 部署到谷歌云，它大概率会开始编。命令敲错、参数猜错、Cloud Run 的配置写歪，最后给你一段看起来很像回事、跑起来全是错的输出。

**问题不在模型不够聪明。它读过全网的文档，但没读过你们这一个项目的部署手册。** 平台越分层，这个落差越大——谷歌云这套 Agent 栈恰好层数不少。

### 三层定位不能混淆

| 层 | 角色 | 解决什么 |
|---|---|---|
| **ADK** | Agent 框架 | 「怎么写 Agent 代码」——管砖头怎么砌 |
| **agents-cli** | 给 coding agent 的规程 + 命令 | 「怎么测、怎么部署、怎么上线」——管脚手架怎么搭、验收怎么过 |
| **Coding Agent**（Claude Code / Codex / Gemini CLI / Antigravity） | 宿主 | 执行规划与写码 |

README 的 FAQ 把这点说死了：**`agents-cli` is a tool *for* coding agents, not a coding agent itself.**

它**不强制**你用 coding agent：整套命令都能脱离 agent 单独跑，Skills 只是让助手更容易替你把流程走完。这一点很重要——它不是一个只能配合特定 IDE 的锁定层。

---

## 2. 核心机制：Agent Skills + 渐进式披露

底层用的是**Agent Skills 开放规范**，文件格式 `SKILL.md`：一个 Skill 一个目录，顶部 YAML frontmatter 写 `name` 和 `description`，正文写具体怎么做。

```yaml
---
name: google-agents-cli-deploy
description: Deploy ADK agents to Google Cloud. Use when asked to deploy.
---
```

### 渐进式披露（progressive disclosure）是关键设计

1. 工具**启动时**不把 Skill 全文塞进上下文，只扫 `name` + `description`
2. 你说了「部署」→ 匹配到对应 Skill → 才加载正文
3. 加载前问一句：**是否允许 `google-agents-cli-deploy` 执行**

这是 token 成本控制的正确做法，也是我这套 skill 体系里「description 前 57 字符必须自包含触发器」的原因——**description 是唯一常驻上下文的部分**。

### 七个 Skill = 一条有先后的流水线

不是并列清单，**有先后之分：先定规矩再起项目，先跑通再上线。**

| Skill | 教你的 agent 什么 |
|---|---|
| `google-agents-cli-workflow` | 整体开发流程、代码保存规则、模型选择 |
| `google-agents-cli-adk-code` | ADK Python API — agents / tools / 编排 / 回调 / 状态 |
| `google-agents-cli-scaffold` | 脚手架 — `create` / `enhance` / `upgrade` |
| `google-agents-cli-eval` | 评估方法论 — 指标、数据集、LLM-as-judge、自适应评分表 |
| `google-agents-cli-deploy` | 部署 — Agent Runtime / Cloud Run / GKE / CI-CD / 密钥 |
| `google-agents-cli-publish` | 注册到 Gemini 平台 |
| `google-agents-cli-observability` | Cloud Trace、日志接入、第三方集成 |

> ⚠️ **文章自己提的安全提醒**：这套机制能减少幻觉，但**挡不住恶意**。公开 Skills 存在一定比例的 prompt injection 风险，装第三方 Skill 前先看来源。这条与我这边的 skill 安全规则同源——**skill 是可以在你的上下文里执行指令的，等同于代码执行权限**。

---

## 3. 装到部署：一条命令一步

### 前置

Python 3.11+、[uv](https://docs.astral.sh/uv/)、Node.js

### 安装

```bash
# 完整安装：CLI + 把 Skills 注入到你的 coding agent
uvx google-agents-cli setup

# 只装 Skills，剩下的让助手自己搞定
npx skills add google/agents-cli
```

安装时会问你要给**哪些** coding agent 注入 Skills，选完把文件写进各自目录，重启助手生效。新增或换工具后：

```bash
agents-cli update   # 强制把 Skills 重装到所有 IDE
```

### 本地开发（完全不需要谷歌云）

```bash
agents-cli create <name> -y     # 默认 adk_python 模板 + agent_runtime 目标
agents-cli run "prompt"         # 单条 prompt 跑一次
```

本地有 **Playground 跑在 8080 端口，支持热重载**。这个阶段用 AI Studio 的 key 就能跑起来，改一行看一行。

官方演示用例很怪但很有代表性：「做一个**穴居人风格的 agent**，把啰嗦的句子压成短促的技术腔」。你不用管目录怎么建、样板代码怎么写——`scaffold` 会生成目录结构、`agent.py`、评估集和 Terraform 配置。

### 评估这一步别跳过

```bash
agents-cli eval run
```

它会跑推理、给**轨迹**打分、生成对比报告。

> 文章原话：大部分 Agent 项目死在「本地跑得好好的，上线就崩」。

### 部署

```bash
agents-cli deploy            # 目标选 Cloud Run / GKE / Agent Runtime
agents-cli deploy --status   # 查当前状态
```

Terraform、IAM 权限、密钥管理都是自动配的。CI/CD 也能顺手搭起来（`agents-cli infra cicd`），保证每次上线前评估自动跑。

**失败时的体验设计**：它会打出构建日志的**最后四十行**，而不是只回一句 `Build failed`。

> 这个细节值得单独记：报错不给线索的时候，agent 只会开始瞎猜——**错误信息的质量直接决定 agent 的自我修复能力**。

---

## 4. 命令按 Agent 生命周期分组（不是按功能堆）

把命令铺开看，能反推设计意图——它按**生命周期**分组：

| 组 | 命令 |
|---|---|
| **登录与准备** | `login`（含 `--status`）、`install`、`lint`（Ruff） |
| **造项目** | `create <name> -y`、`scaffold enhance`（给老项目补部署/CI-CD）、`scaffold upgrade`（升版本） |
| **跑和评**（最长的一组） | `run`、`eval run` / `generate`（只跑推理）/ `grade`（只打分）/ `compare`（两次结果对比）/ `analyze`（失败模式聚类）/ `optimize`（**用评估数据反调提示词**）/ `dataset synthesize`（造多轮测试集）/ `metric list` |
| **发布** | `deploy`、`publish gemini`、`infra single-project`、`infra cicd` |
| **维护** | `info`、`update` |

### 两个改名（老教程里的写法已失效）

| 旧写法 | 新写法 |
|---|---|
| `publish gemini-enterprise` | `publish gemini` |
| `scaffold <项目名>` | `create <项目名>`（`scaffold` 只留给 `enhance` / `upgrade`） |

---

## 5. 版本与变更（v1.11 → v1.12）

文章成稿在 v1.11.0 附近，其中两处变化值得记：

1. **新项目默认用 Agent Identity 作为运行身份**，不再是 service account，部署出来的 Terraform 也跟着走。
2. **v1.11 有破坏性变更**：`adk` 加模板名的快捷写法被移除，现在要把 **recipe 的 GitHub 地址**传给 `create`。

其余演进：
- 模板不再只有 ADK：LangChain 的 agent 现在按项目名命名；ADK Go 模板补了构建重试与超时丢回复的问题
- 新项目默认依赖 `google-adk >= 2.12`
- **扩展能带 Skill 了**：`extension add` / `update` / `remove`，可以把一套 Skill 装到项目里或全局——团队自研能力就这样跟着扩展走

### 已知短板（文章直说）

- 远程服务器上做鉴权比较麻烦，经常要手动传 token
- 早期版本做复杂多 Agent 编排时，**延迟能到 60–90 秒**
- Skill 能减少幻觉，**不能消除**——指令太模糊它照样按自己理解走

---

## 6. 值得抄的设计（比功能更长期的部分）

### ① 真正的亮点是 eval 这条线，不是部署

> 「轨迹评分这个思路指向一个方向：以后测 Agent 除了看答案对不对，还要看**推理链走没走对**。它甚至给了 `optimize` 命令，用评估数据反过来调提示词，等于把『评估』变成了『迭代的输入』而不是『上线前的考试』。」

这条判断我认为是对的，而且比文章其它结论更硬：**eval 从 gate 变成 loop input，是 Agent 工程化真正分水岭**。`eval analyze`（失败模式聚类）+ `eval optimize`（反调提示词）这套闭环，比「上线前跑一次评估」有价值一个数量级。

### ② 把云平台的隐性知识显式化

它提升的不是模型智力，是**把散落在文档里的部署知识搬进了上下文**。同样一个模型，读过你们项目的部署规程和没读过，交付质量是两个量级。

对应到本地实践的启示：**Skill 的本质是把「组织内隐性知识」编译成模型可执行的形式**，而不是把通用知识再抄一遍。

### ③ 一句话的部署决策

文章最后的建议很克制，值得直接采纳：

> **如果团队只在本地折腾，先只装 `eval` 那一个 Skill 也够用。**

---

## 7. 与我现有体系的关系 / 判断

| 维度 | 判断 |
|---|---|
| **要不要装** | **暂不装**。需要 Python 3.11 + uv + Node，且真正价值在部署到谷歌云；无 GCP 项目时只是多一堆 Skill 占上下文 |
| **值得单拿的部分** | `google-agents-cli-eval` 的方法论（轨迹评分 / LLM-as-judge / 自适应评分表）+ `eval analyze`/`optimize` 的闭环思路 |
| **与已有归档的关系** | 承接 [`AI兵器谱47-ADK-Python-Google-Agent开发套件`](2026-08-01-AI兵器谱47-ADK-Python-Google-Agent开发套件.md)：ADK 管「怎么写」，agents-cli 管「怎么测和怎么上线」，**两篇是上下游** |
| **与 skill 体系的关系** | 渐进式披露 + frontmatter `description` 做触发的做法，与我现有 skill 体系同构；差异在于它**装完会问是否允许该 Skill 执行**（显式授权点），这个设计值得借鉴 |
| **风险** | 第三方 Skill = 第三方代码在上下文里执行，prompt injection 面；`AGENTS_CLI_TELEMETRY` 默认上报使用元数据（可用 `DO_NOT_TRACK=1` 或 `AGENTS_CLI_TELEMETRY=0` 关掉） |

### 遥测（README 原文事实）

默认每次命令调用发一个使用信号，写入**你自己的** GCP 项目日志（`logName=projects/<project>/logs/agents-cli`），只记命令名、退出码、错误类、耗时、CLI/Python 版本、OS——**不含文件内容、参数或其它输入**。但请求头（`User-Agent` / `X-Goog-Api-Client`）里的用量元数据 Google 会收到。关闭方式：

```bash
export AGENTS_CLI_TELEMETRY=0
# 或标准变量
export DO_NOT_TRACK=1
```

查当前状态：`agents-cli info`

---

## 8. 关联链接

- 仓库：https://github.com/google/agents-cli
- 文档：https://google.github.io/agents-cli/
- PyPI：https://pypi.org/project/google-agents-cli/
- Release Notes：https://github.com/google/agents-cli/blob/main/RELEASE_NOTES.md
- ADK：https://adk.dev
- 邻近 skill 套件（README 推荐搭配）：[addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)（通用软件工程工作流）、[google/skills](https://github.com/google/skills)（GCP 基础：BigQuery / Cloud Run / Firebase / GKE）
- 原文（Java基基）：https://mp.weixin.qq.com/s/P5jBCwbxDoibeF_23tw9YQ

---

**归档日期**：2026-10-30 ｜ **归档人**：幕僚长 ｜ **核实来源**：GitHub API（仓库元数据 / releases 列表）+ 仓库 README 全文 + 微信原文正文
