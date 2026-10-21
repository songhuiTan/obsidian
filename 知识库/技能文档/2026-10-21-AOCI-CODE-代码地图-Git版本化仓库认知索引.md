# AOCI-CODE：给代码库建一张 Git 版本化的「认知地图」，Agent 动手前先读一遍

> 来源：GitHub 开源项目 [aoci-spec/aoci-code](https://github.com/aoci-spec/aoci-code)｜官网 [aoci.ai](https://www.aoci.ai)
> 归档日期：2026-10-21（GitHub API 实测：1,669 ⭐ / 200 forks / 38 open issues / Go / 2026-08-08 建库 / 2026-10-16 最后推送 / v0.1.0-rc21 / FSL-1.1-MIT）
> 标签：AOCI, 代码地图, Code Map, MCP, 仓库认知, 索引治理, CodeGraph, 上下文工程

---

## 概述

一句话摘要：AOCI = **AI-Oriented Cognition Infrastructure**，把"整个代码库是干什么的"写成一份**纯文本、Git 版本化、受治理的索引**，Agent 动手前先读它一遍，此后不必每个任务都重新翻仓库。

**问题根源（和 08-07 那篇代码图谱一致）**：LLM 无状态，每次进仓库都从零推导谁调谁、改这行会波及什么，只能靠最贵的办法补齐——不停 `Read`/`Grep`/`Glob`。代价不只是钱：上下文被无关文件撑满后开始编造不存在的依赖关系。

**AOCI 的答案与 CodeGraph 不同**：不是"问什么查什么"，而是"**先读完，再动手**"。

## 核心定位：与 CodeGraph 的分工

README 自己划了界线：

| | CodeGraph 类工具 | AOCI-CODE |
| --- | --- | --- |
| 产出 | 符号与调用关系的图 | 每个文件/每张表**一行模型撰写的条目**：它负责什么、连着什么、什么不能坏 |
| 时机 | 任务中按需查精确源码和调用路径 | 任务开始前**读完整索引**，已知系统全貌 |
| 形态 | 查询服务 | 仓库内的纯文本文件，Git 可 diff / review / 回滚 |
| 定位 | 俯身取证的放大镜 | 抬头看路的地图 |

**官方推荐组合**（原文）：AOCI-CODE 提供**全局语义先验**，代码图谱 / LSP / 搜索 / 源码提供**精确证据**，测试与运行时结果提供**最终验收**。

同时 README 明确否认自己是：AI Agent、RAG 替代品、AST 替代品、CodeGraph 替代品、向量库、常驻 daemon、云服务。**Agent 和结构工具仍然要读真实源码和测试。**

## 索引结构：四个 Volume，一份 Whole-Index

索引是项目里的纯文本文件集，跟着代码走、被 Git 版本化：

```text
aoci.txt              Root：声明当前 CognitionSet 与参与的 Volume（激活入口）
aoci.meta.txt         Meta：标签字典、FRAS 规则、配额、撰写契约
aoci.code.txt         Code：代码与仓库资产的条目
aoci.database.txt     Database：可选，表级条目（开启 Database Cognition 后才有）
```

- **Root 最后发布**——这样"部分资产"永远不会被误当成"完整索引"。这是索引治理里最实用的一条工程约束。
- Fresh 项目直接使用 **Code-only Volumes** 布局；老仓库走 Legacy 迁移（`aoci cognition plan` 只读预览迁移计划）。

### FRAS 四字段：一条条目要回答什么

| 字段 | 回答的问题 |
| --- | --- |
| **F — Function** | 这个对象的核心职责是什么 |
| **R — Relationships** | 哪些必须放在一起才能理解的强关系 |
| **A — Admission/Contract** | 它的外部接口或契约是什么 |
| **S — Non-obvious constraints** | F/R/A 之外**不显然**但重要的信息（例：原生发布不得退化为覆盖式重命名；保留第三方状态；失败时 fail closed） |

S 字段是整套设计的精髓——它记录的是**从代码结构里推不出来的东西**：事务边界、兼容约束、什么不能碰。这也正是"再读一遍源码"永远补不齐的部分。

## 9 个 MCP 工具（stdio，无 daemon）

| 类别 | 工具 |
| --- | --- |
| 读 | `aoci_rules`、`aoci_overview`、`aoci_get_entries`、`aoci_search` |
| 维护 | `aoci_maintain`、`aoci_update_entry`、`aoci_remove_entry` |
| 证据 | `aoci_header`、`aoci_report` |

`aoci mcp` 不需要常驻进程；stdout 保留给协议，日志走别处。工具名、schema 和状态值以二进制本身为准，README 只是起点。

## 三阶段工作流

### 1. 受治理地建索引

```bash
# 从源码构建（当前为 RC，需签名包或源码构建）
git clone https://github.com/aoci-spec/aoci-code.git
cd aoci-code && go mod verify && make build
./build/aoci --version

# 初始化 + 扫描
aoci --repo . init --locale en-US --agent codex
aoci --repo . scan
```

- `init` 只创建 Root、Meta 和空的 Code Volume，**从不替你编造仓库语义**。
- `scan` **不可跳过**——它建立后续所有步骤绑定的 **Baseline**。跳过则 Guide 直接报 `blocked` + `baseline_missing`，没有任何写作路径。只有 scan → Guide → 无参数 Maintain 这一条路。
- 真正的语义由宿主 Agent 按 Guide 批量写入（`aoci_update_entry`），完成后 `verify` + `check`。

### 2. 动手前先读

Agent 读项目规则 → 当前 Guide → 完整 Whole-Index，然后才去看源码和证据。

**Overview 分块投递有完整性证明**：超过 chunk 预算时从 chunk 1 开始，Agent 必须自动跟随 `next_cursor` 走到最后一块，再提交一次 attestation。原文措辞很硬——不得把"本地搜索、旧记忆、补充阅读或直接读文件"冒充成完整投递；快照不一致就 fail closed，绝不混内容。

### 3. 验证性变更后维护

代码和测试稳定后，由项目规则 + MCP 工作流让 Agent 更新受影响的条目，把索引拉回 `aligned`。MCP server 检测代码变更并**派发**待更新条目——所以"建完就腐坏"这个最大风险被流程化处理了。

## CLI 速查

| 命令 | 用途 |
| --- | --- |
| `aoci --repo . init` | 初始化 Volumes（不产生语义） |
| `aoci --repo . scan` | 建立 Baseline（强制） |
| `aoci --repo . verify` / `check` | 校验对齐（不改正式索引与 Baseline） |
| `aoci --repo . doctor` | 环境/能力诊断 |
| `aoci database ...` | 显式配置并验证 PostgreSQL / MySQL / openGauss schema 证据 |
| `aoci database source access` | 只读检查外部是否提供了凭证引用，**不返回凭证值** |
| `aoci database cognition bootstrap` | 给已对齐的 Code-only 项目加 Database Cognition |
| `aoci cognition plan` | 只读预览 bootstrap / Legacy→Volumes 迁移计划 |
| `aoci cognition system lineage` | 派生 System Cognition 观测 |

`status --deep`、`index score`、`index agent plan` 是 **Legacy-only** 命令，不是新项目路径。

## 三种执行模式

| 模式 | 数据流向 |
| --- | --- |
| **Agent-native** | Codex / Claude Code / Cursor 等 MCP 宿主生成语义，AOCI 治理语义 |
| **Endpoint-native** | 可选 AI 层只把源码发给用户显式配置的端点 |
| **Deterministic-only** | 关掉 AI，只用扫描、校验、查询和 CI |

## 规模与边界

- 设计上限约 **50 万行**；官方称一个 **70 万行**的商业系统在用，索引约 **30 万 token**。
- 真正的限制不是行数，**是索引体积**。
- 索引放在仓库里 → 换人、换 Agent、换会话，读一次就接上，不用重建理解。
- 本地优先：只读源码与库表结构，不碰业务数据，不联网、不上传；库凭证只按环境变量名引用，从不存储。唯一对外连接是你声明的数据库（取目录元数据）和它自己的 loopback 状态页。

## 反例与风险（必须一起看）

- **RC 阶段**：当前 `v0.1.0-rc21`，需从源码构建或下载签名包并做发布校验；不是 `npm i` 一条命令的事，需要 Go 工具链（`go.mod` 声明版本）。
- **不是 OSI 开源**：FSL-1.1-MIT 属 Fair Source / source-available，商用与再分发条款要自己读 LICENSE。
- **索引吃上下文**：70 万行 → 30 万 token。小项目上来就建索引未必划算，收益随代码量增大才显著。
- **语义质量取决于写它的 Agent**：索引是模型逐条撰写、协议治理的产物，写歪了是一份"被治理得很好的错误地图"。必须靠 verify/check 和读源码兜底。
- **宿主兼容性看协议不看模型**：需要支持标准 stdio MCP 并能遵守工具契约的宿主；单看模型名字不能判断可用。

## 战略分析

**与前序归档的关系**：本文补上了知识库里 codeMap 谱系缺的那一块——08-07 的 sense/codegraph 解决"精确查询"，Archify 四篇解决"看得见的图"，06-16 九张图解决"用提示词批量生成"，而 AOCI 解决"**地图本身作为受治理资产长期存在**"。

**与 Hermes 生态对照：**

| 维度 | AOCI-CODE | Hermes 现有能力 |
| --- | --- | --- |
| 先验式系统认知 | Whole-Index 先读后做 | 无（每次靠 `read_file`/`search_files` 现场重建） |
| 精确符号查询 | 交给 CodeGraph/LSP | gbrain `code_def`/`code_callers`/`code_callees`/`code_refs` |
| 影响面分析 | 索引里写明"什么不能坏" | gbrain `code_blast`（分层 transitive callers）/ `code_flow` |
| 地图的版本治理 | Git 化 + diff/review/回滚 + Root 最后发布 | 无对应层 |
| 防腐坏机制 | MCP 派发待更新条目 + `aligned` 状态机 | 无对应层 |
| 数据边界 | 只读、本地、不联网 | 本地天然符合 |

**最值得借鉴的三条设计（与工具是否采用无关）：**

1. **"Root 最后发布" + 原子写入 CAS + fail closed** —— 任何"部分完成"的资产都不能被误认为整体。这条规则可以直接搬到知识库索引：README / 快速导航 / 索引目录三者任何一方没更新完，就不算归档完成。
2. **索引住在被测对象内部，而非外部服务** —— 跟着仓库走才能 diff、review、回滚，才能"换人换会话不掉线"。外部 MCP 服务一旦停摆，认知层就消失。
3. **S 字段专门记录不显然的约束** —— "什么不能坏"比"它是什么"更难从代码推导，也更值钱。项目文档里最缺的正是这一栏。

**判断**：AOCI 的方向不是"更快的 grep"，而是把**仓库认知变成第一类资产**（可版本化、可审查、可回滚、可交接）。与小谭现有"知识库 wiki 化"是同一命题在不同对象上的应用——一个是给知识建地图，一个是给代码建地图。

## 关联阅读

- [[2026-08-07-给ClaudeCode装代码图谱-工具调用降47-Token降32]] — sense/codegraph/CodeGraph 三工具与量化基准（同主线：认知前置）
- [[2026-08-16-archify-代码库生成可交互架构图-极客之家]] — 读代码库出可交互架构图，模型出 JSON、程序渲染
- [[2026-06-16-9张AI生成的图吃透任何一个前端项目]] — 九段提示词批量产出代码地图（零安装路线）
- [[OpenWiki-Agent时代代码库文档基础设施]] — 代码库→面向 agent 的文档，`openwiki --init/--update`
- [[2026-06-05-OpenViking给Agent建可检索会生长的上下文数据库]] — L0/L1/L2 三层上下文分层架构

## 归档日志

- 2026-10-21 归档。事实来源：GitHub API 仓库元数据 + README + docs/getting-started.md（均以 v0.1.0-rc21 为准）。未做安装实测，未验证索引生成效果——本条目为资料归纳，不含实测结论。
