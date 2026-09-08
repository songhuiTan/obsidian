# 给 Claude Code 装代码图谱：工具调用降 47%、Token 降 32%，附 3 个开源工具对比

> 来源：码哥｜码哥跳动 2026-08-07
> 原文：[给Claude Code装代码图谱，工具调用降 47%、Token 降 32%，附 3 个开源工具对比](https://mp.weixin.qq.com/s/s2fgHY8ndmgef3WljoWYpQ)
> 标签：代码图谱, Code Graph, MCP, Claude Code, sense, codegraph, Token优化, AI编程

---

## 概述

一句话摘要：给 Claude Code 接一张本地代码地图，工具调用最多降 47%、Token 降 32%，AI 不再反复翻读源码。

**问题根源**：LLM 是无状态概率模型，每次新会话都像第一次进仓库，要从零推导谁调用了谁、改这一行波及哪些测试。没有代码库的「心智地图」，只能靠最贵的办法补齐认知——不停地 `Read`/`Grep`/`Glob`。代价不只是钱：上下文被无关文件撑满后还会产生幻觉，编造根本不存在的依赖关系。**缺的不是更聪明的模型，而是一层持久化的代码结构认知。**

## 三工具速查表

| 你的诉求 | 推荐工具 | 关键理由 |
|---------|---------|---------|
| 想要最硬的量化证据 | **sense**（luuuc，25 stars） | 公开 A/B 基准：工具调用 -47%、Token -32% |
| 想要零维护、自动同步 | **codegraph**（colbymchenry，6.4 万 stars） | 改文件自动更新图谱，索引永不过期 |
| 想要全家桶 + CI 评审 | **CodeGraph**（codegraph-ai，51 stars） | 45 个 MCP 工具，PR 自动出分析评论 |

⚠️ **GitHub 两个重名仓库必须分清**：`colbymchenry/codegraph`（6.4 万 stars 本地图谱）vs `codegraph-ai/CodeGraph`（51 stars 全家桶），装错白忙活。

## 代码图谱原理

**核心思路：把「理解」前置到索引阶段。** 平时没人问时先把代码库嚼碎存成结构化关系网，Agent 需要时直接查，而不是现场重新推理。

数据流：源码 → `tree-sitter` 解析 → 抽出符号与关系（调用者/被调用者/继承链/测试归属）→ 落成图谱 + `bi-encoder` + `cross-encoder` 语义 embedding → 通过 MCP 暴露给 Claude Code/Cursor 等 Agent。

**关键：图谱是确定性的结构查询，不是概率猜测。** Agent 问「谁调用了 place_order」，图谱直接返回精确调用列表。

**边界（反例）**：纯图谱不能替代语义搜索——「这个项目里处理超时重试的惯例是什么」这种模糊意图要靠 embedding。主流工具都是图谱 + 向量双管齐下。

## 重点实测：sense（量化证据最硬）

基准环境：Claude Code Opus 4.6，跨 7 个真实仓库（Discourse、Flask、Next.js、Axum、Gin、Javalin + 私有电商），规模和语言分散。

| 指标 | 接图谱前 | 接图谱后 | 变化 |
|------|---------|---------|------|
| 工具调用/任务 | 19 次 | 10 次 | **-47%** |
| Token/任务 | 228K | 156K | **-32%** |
| 成本/任务 | $0.42 | $0.31 | -26% |
| 会话时长 | 91s | 73s | -19% |
| 每 100K token 得分 | 0.19 | 0.30 | **+64%** |
| 每分钟得分 | 0.28 | 0.38 | +37% |

质量没掉：同正确性下工作量大幅下降——这才是「减法革命」的真义。

**sense 只暴露 4 个 MCP 工具**，对应 Agent 改代码最频繁的四类诉求：
- `sense_graph`：查符号关系和调用链、死代码
- `sense_search`：bi-encoder + cross-encoder 混合语义搜索
- `sense_blast`：算影响面和风险分
- `sense_conventions`：从源码检测项目规范

**性能**（自身仓库 382 文件/4032 符号）：图查询 p50 0.2ms、p95 3ms；影响面 p50 0.1ms、p95 10ms；冷启动 48ms；全量扫描 4.9s、增量 2.3s。

**反例**：仅 25 stars 生态小；语言支持 13 种双 tier（比另两个少）；本地索引占磁盘（100-200MB）。

## 另外两个工具

### codegraph（colbymchenry，6.4 万 stars）— 自动同步派

- Rust 写，100% 本地，连 Node.js 都不需要；pre-indexed code knowledge graph
- `codegraph init` 生成 `.codegraph/` 构建完整图谱，之后**自动监听文件变更，索引永不过期**
- 安装：`curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh` 或 `npm i -g @colbymchenry/codegraph`
- 支持 30+ 种语言（TS/Python/Go/Rust/Java/C#/C/C++/Ruby）
- 无公开标准 A/B 基准，需自测

### CodeGraph（codegraph-ai，51 stars）— 全家桶派

- C 语言写，45 个 MCP 工具（README 写 45、描述写 42），37 种语言走 tree-sitter
- VS Code 扩展 + 持久记忆层（跨会话代码认知，第二次进项目不用重新建图）
- `--graph-only` 标志跳过 embedding：索引快 10-50 倍、不需要 API key，适合 CI 或一次性查询
- GitHub Action：每个 PR 自动出代码图谱分析评论（影响面、测试缺口、过时文档、建议评审人）
- `codegraph-rules-for-agents` 仓库：预置规则教 Agent 优先用 `codegraph_*` 工具而非裸 grep
- 接入：`~/.claude.json` 写 `{"mcpServers":{"codegraph":{"command":"/path/to/codegraph-server","args":["--mcp"]}}}`

**反例**：45 个工具的复杂度不是谁都吃得消，小团队容易陷在配置里。

## 动手接入：5 分钟给 Claude Code 装 sense

```bash
# 第一步 安装
curl -fsSL https://luuuc.github.io/sense/install.sh | sh
# 或 go install github.com/luuuc/sense/cmd/sense@latest
sense --version   # ✅ 验证二进制进了 PATH

# 第二步 扫描项目（tree-sitter 解析 + ONNX embedding，写进本地 .sense/）
cd your-project
sense scan        # ✅ 验证：目录出现 .sense/，后续增量更新

# 第三步 接入 Claude Code（自动探测 Claude Code/Cursor/Codex CLI/OpenCode）
sense setup       # ✅ 验证：根目录出现 .mcp.json，CLAUDE.md 多了 sense 路由
```

接好后让 Claude Code 查「改 place_order 会影响哪些测试」，`sense_blast` 直接返回影响面清单。新会话确认 4 个工具（sense_graph/search/blast/conventions）出现即通。每天跑一次 `sense scan` 增量更新（2.3s）即可。

**常见报错**：
- `sense: command not found` → `$GOPATH/bin` 加进 `.zshrc`
- `go install` 版本不匹配 → Go 需 ≥1.21
- 旧版残留 → 先 `sense uninstall` 清干净，避免 `.sense/` 旧索引串味

## 自测方法（codegraph 和 CodeGraph 无公开基准）

1. 定一组固定任务：「找出调用 X 的所有地方」「改 Y 会影响哪些测试」
2. 对比接图谱前后的回答正确性和引用文件数
3. 工具调用数：Claude Code `/status` 或 codeburn（监控 31 个 AI 编程工具 token/成本）
4. Token 数：`/cost` 命令或 codeburn
5. **坑**：别只看数字下降——调用数降了但答案出错 = Agent 偷懒过头，要调 MCP 提示词或索引范围。每次改动后跑同一组固定任务做回归，把「调用数、Token、正确率」三栏一起记

## 局限与选型建议

- 索引代价：本地毫秒级查询是真，但索引要占磁盘（sense 382 文件 ≈ 100-200MB）；大仓库首扫按规模增长，选空闲时跑
- 语义搜索依赖 embedding 质量，cross-encoder 翻车时图谱结构化关系兜底，但模糊意图仍可能失准
- **选型三判断**：小项目值得上但收益随代码量增大才显著（先上 sense 零配置最划算）；要零维护选 codegraph；要团队 CI 评审选 CodeGraph（前提接受 45 工具复杂度）
- **隐私**：三主推工具全 100% 本地、无 API key，源码不出本机；金融/政务合规场景本地天然优于云端 MCP

## 常见问题

- **图谱 vs RAG/向量检索**：RAG 是语义块检索给相似文本让你猜；图谱是结构化关系网（调用者/被调用者/继承/影响面都是确定边）。实际工具两者并用，图谱管结构、向量管模糊语义，不是替代关系
- **接了图谱会变慢吗**：不会，查询毫秒级；省下的翻文件时间远大于查图耗时

## 关键洞察

代码图谱不是又一个花活，是 AI 编程正在发生的**减法革命**：让 Agent 从「每次从零推导」变成「读一张持久地图」，token、工具调用、幻觉率一起往下掉。**2026 年给编程 Agent 配本地代码地图会从可选项变成标配——就像当年 linter 从奢侈品变必装。**

---

## 战略分析

**与 Hermes 体系的直接对照：**

| 维度 | 本文三工具 | Hermes 已有能力 |
|------|-----------|----------------|
| 符号关系查询 | sense_graph / codegraph | gbrain `code_callers`/`code_callees`/`code_def`（tree-sitter 调用图） |
| 影响面分析 | sense_blast / CodeGraph PR 评论 | gbrain `code_blast`（transitive callers 分层）/ `code_flow`（entry→sink） |
| 语义搜索 | sense_search（bi+cross encoder） | gbrain `query`（混合检索 + 多查询扩展） |
| 引用查找 | codegraph | gbrain `code_refs`（含注释/字符串全量引用） |
| 项目规范检测 | sense_conventions | Hermes AGENTS.md / CLAUDE.md 静态配置 |
| PR 自动评审 | CodeGraph GitHub Action | 已归档的 Claude Code 生产栈（GitHub Action 自动 PR review + confidence 过滤） |

**核心发现：gbrain 已内置本文主张的「代码图谱」能力**（code_blast/code_flow/code_callers 等 7 个结构化代码查询工具，tree-sitter 调用图 + 深度遍历），架构上比 sense 的 4 工具更全。差距在于：gbrain 目前是 MCP 工具供 Agent 按需调用，**没有做「自动索引 + 增量同步」的常驻层**（sense 的 scan/setup 生命周期、codegraph 的文件监听）——这是可以借鉴的工程化方向。

**对用户的价值**：
1. 如果希望 Hermes/Codex 在本地大仓上减少翻文件，可以给 Codex 接 sense（Go 单二进制、`sense setup` 自动写 `.mcp.json` 探测 Codex CLI）——与之前"增强 Codex 搜索 GitHub"的需求互补（那个管外部仓库，这个管本地代码理解）
2. 「图谱管结构、向量管模糊语义」双管齐下原则，与 gbrain 的 hybrid 检索设计一致
3. 自测方法论（固定任务 + 三栏回归记录）可复用于任何工具接入评估

**一句话**：代码图谱 = 把「理解」前置成可查询的确定性地图，是 AI 编程 token 优化的下一个标配；Hermes 生态里 gbrain 已具备核心查询能力，缺的是自动索引层。

## 归档日志

- 2026-08-07 归档
