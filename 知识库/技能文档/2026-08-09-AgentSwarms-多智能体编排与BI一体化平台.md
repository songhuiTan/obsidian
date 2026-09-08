# AgentSwarms：18 种节点 + 27 个连接器，把多智能体编排和 BI 仪表盘做成同一套系统

> 来源：如此才是 2026-08-09
> 原文：[源码可得、自己部署：18 种节点 + 27 个连接器，AgentSwarms 把多智能体编排和 BI 仪表盘做成了同一套系统](https://mp.weixin.qq.com/s/9xmJ37EoqFZrnbCro-gqkQ)
> GitHub：[AgentSwarms-fyi/agentswarms](https://github.com/AgentSwarms-fyi/agentswarms.git)
> 标签：AI, Agent, 多智能体, BI, 商业智能, 自部署, ELv2, LangGraph, DuckDB, MCP

---

## 概述

**AgentSwarms：同一套 UI 跑两套使命。** 仓库版（ELv2 许可）面向"在自有基础设施上跑平台"，治理、BYOK、SSRF、IAM、Trace 全部自己管；agentswarms.fyi 是托管学习版。

本文只讲仓库里能验证的东西：**能力边界、三种装法、六个开关、执行器内部。**

## 能力全景（构建 / 数据 / 智能 / 治理四象限）

- **画布**：18 种节点（入/出/智能体/路由/条件/循环/审批/评估/A2A）
- **Agent Chat**：Visual BI 同会话出图，PPTX/Word/Excel 输出，可嵌入 iframe
- **27 连接器**：22 个仓库 + 5 个 SaaS，只读 + AES-GCM 加密
- **知识库**：Drive/Notion/SharePoint/Dropbox 同步，两级去重，每源 500 项、40 万字
- **BI + AI 分析师**：18 种可视化 + SQL 下推 + 行/列遮蔽；NL→SQL 在 DuckDB 上 75.4%
- **IAM + 共享**：deny-by-default、模型白名单、行/列遮蔽、SAML SSO
- **14 家模型 + 17 个工具**：`metric_query` 默认拒答；`web_browse` 需 Firecrawl
- **MCP Builder**：FastMCP → 沙箱 → Streamable HTTP；工具变更强制重新批准

## 安装：三种装法

### 1. 一键脚本

```bash
git clone https://github.com/AgentSwarms-fyi/agentswarms.git
cd agentswarms
cp .env.example .env   # 填三个 Supabase key
bash scripts/setup.sh --all   # app + docgen + notebooks + sandbox
```

脚本自动生成凭据、推 150 个迁移、起 Compose。`--all` 是完整产品，docgen 带 LibreOffice（约 1GB），按需开关。

### 2. 源码开发模式

```bash
npm install
npx supabase login && npx supabase link --project-ref <ref> && npx supabase db push
npm run dev   # http://localhost:8080
```

Supabase 控制台把 Site URL 与 Redirect URLs 都设成 `http://localhost:8080`。改 `.env` 必须重启 dev server。

### 3. 源码构建镜像

```bash
docker build \
  --build-arg VITE_SUPABASE_URL=... \
  --build-arg VITE_SUPABASE_PUBLISHABLE_KEY=... \
  --build-arg VITE_ADMIN_EMAIL=ops@example.com \
  -t agentswarms:1.0 .
```

`VITE_*` 构建期烘焙，service-role / OPENROUTER key 运行时读。⚠️ Windows `npm run build` 在 cmd.exe 报 `'NODE_OPTIONS' is not recognized`——用 Git Bash / WSL2，或手动 `set NODE_OPTIONS=--max-old-space-size=6144 && npx vite build`。

## 高效使用：六个开关

1. **Web 搜索**：不配 key 走 DuckDuckGo；粘 Firecrawl key 升级。SSRF 挡云元数据
2. **共享连接以 owner 凭证运行**：secret 不离开服务端；行/列遮蔽强制
3. **SQL 聚合下推**：仓库表 `linked live` 时编译为仓库方言 SQL
4. **语义层选名字不写 SQL**：`metric_query` 按 IAM 白名单强制
5. **Function 节点沙箱化**：`--profile sandbox` 起独立容器，避免 service-role key 暴露
6. **多实例关掉自带调度**：`DISABLE_INPROCESS_SCHEDULER=1` + 外部 cron 命 `/api/bi/cron`

## 技术架构

**栈**：TanStack Start（React 19）+ Supabase（Postgres + Auth + Storage + pgvector）+ LangChain/LangGraph + XYFlow + DuckDB-Wasm + pptxgenjs/docx/write-excel-file。自托管下 Kong 做 API 网关；PWA 不缓存 `/api/*`。

### 执行器内部（核心细节）

引擎从 `src/lib/swarmGraph.ts` 开始：**画布运行时与无头执行器共用同一份图语义**，`swarmExecutorParity.test.ts` 守住边界，避免"画布能跑、部署翻车"。

- **拓扑分层**：自环视为迭代；同层并发按层序提交（`SWARM_LEVEL_CONCURRENCY` 默认 4）
- **检查点**：每层一次，`swarm_run_checkpoints` 按 `run_id` upsert 原地覆盖
- **人审节点**：`suspension.at` 挂在层上，层结束整次 run 进 `suspended`；`approvals` 插 `action_type='swarm_step'`；已批准节点必须 resume 重入拿决策
- **错误处理**：`onError` 默认 `fail`，条件/路由永远 fail close；重试 5 次钳制 30s，整次 run 600s wall-clock，subswarm 继承父预算

**BYOK 加密**：Web Crypto + `PROVIDER_CREDS_SECRET` 经 SHA-256 派生 256bit 密钥 + 12 字节 IV；`openssl rand -hex 32` 生成，轮换使旧凭据全部失效。

**工具注册表**（1688 行）：定义 + handler + gate；`buildRoutingGuidance()` 按实际启用工具动态拼 TOOL ROUTING 提示词。

**数据仓库**：按 `config.provider` 切换驱动；语句以 `SELECT/WITH/SHOW/DESCRIBE/EXPLAIN` 开头护栏；SSRF 默认挡云元数据。

## 关键结论

把 agentic AI 与 BI 当一整件事做：**画布真能跑、部署真能续、人审真能挂起、SQL 真能下推、凭据真加密。**

自托管 + BYOK + 多智能体 + 数据仓库 + 仪表盘 + Trace 同时压过来的人可以列入候选——前提是接受：ELv2 许可、150 个迁移落库、约 44% 数据/BI 代码无测试要自己补。

---

## 战略分析

**AgentSwarms 与知识库已归档内容形成多线对照，是"多智能体编排 + 数据闭环"的完整参考实现：**

| 本文能力 | 知识库已有对应 |
|---------|--------------|
| 画布 18 节点 + 执行器拓扑分层 | 多 Agent 编排（AgentSpace/AgentScope/WorkBuddy）、LangGraph 讲透系列 |
| 人审节点挂起 + approvals 重入 | HITL（Hook 链护栏、三类生产问题） |
| SQL 下推 + 行/列遮蔽 + deny-by-default | 生产级企业知识库指南（权限到 chunk 级别、检索阶段过滤） |
| BYOK 加密 + secret 不离开服务端 | Agent 治理（越权拦截、权限隔离） |
| 工具注册表 + 动态路由提示词 | 阿里 SkillWeaver（跳过加载全部工具）、技能路由 |
| MCP Builder + 工具变更重新批准 | MCP 生态、Agent 工具治理 |
| 无头执行器与画布共用图语义 + 一致性测试 | Harness 工程（画布能跑、部署翻车 → 测试守住边界） |

**核心启发：**

1. **"画布运行时与无头执行器共用同一份图语义"是本文最值得抄的设计**——可视化编排与实际执行不分离，配合 parity 测试守住边界，直接解决"demo 能跑、生产翻车"的普遍问题。这与 Hermes 的"Skill 测试验证"思路同构。

2. **执行器细节都是生产级硬经验**：fail close（条件/路由永远 fail 而非静默通过）、重试 5 次钳制 30s、整次 run 600s wall-clock、subswarm 继承父预算——这些数字是可直接复用的编排护栏配置。

3. **人审节点挂起的实现**（suspension.at 挂层上、已批准节点必须 resume 重入拿决策）比简单 interrupt 更严谨——决策不能"补批"，必须重跑上下文。对 HITL 设计有直接参考价值。

4. **"多智能体编排 + BI 同套 UI"是差异化定位**——大多数框架只管 Agent 编排，AgentSwarms 把数据仓库、BI 仪表盘、Trace 纳入同一闭环。对要做"数据驱动 Agent 平台"的团队是完整蓝本。

5. **局限明确**：ELv2 许可（非纯开源）、150 迁移、44% 代码无测试——适合作为参考架构而非无脑接入。

**一句话**：AgentSwarms = 多智能体编排 + 商业智能一体化平台，执行器细节（fail close/人审重入/预算继承）和"画布执行器共用图语义"设计是核心价值，适合作为自托管 Agent 平台的架构蓝本。

## 归档日志

- 2026-08-09 归档
