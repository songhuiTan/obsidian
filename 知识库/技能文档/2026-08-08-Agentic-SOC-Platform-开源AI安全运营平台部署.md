# 部署了一个开源 AI 安全运营平台：告警自动收敛、LLM 研判、Agent 直接办案

> 来源：如此才是 2026-08-08
> 原文：[部署了一个开源 AI 安全运营平台：告警自动收敛、LLM 研判、Agent 直接办案](https://mp.weixin.qq.com/s/pTSNZ6PLl_7fgLePYwq2KQ)
> GitHub：[FunnyWolf/agentic-soc-platform](https://github.com/FunnyWolf/agentic-soc-platform)
> 标签：AI, Agent, 安全运营, SOC, SOAR, LLM研判, Harness Agent, 开源, Django

---

## 概述

安全运营痛点：SIEM 每天吐出几万条告警，人力只能逐条看几百条。漏报、误报、重复报混在一起，真正该深挖的线索反而被淹没。

**Agentic SOC Platform（ASP，MIT 协议开源）**：让告警先收敛成可调查的 Case，再由 LLM、Playbook 和 Harness Agent 一起完成研判、富化和知识沉淀。可私有化部署，源码对二次开发友好。

定位：**不是另一个 SIEM，也不是单纯的 SOAR**——更像一个"告警加工厂"：把上游杂乱无章的告警流，加工成结构化、可解释、可行动的 Case。

## 一、10 个功能点

1. **告警收敛成 Case**：上游支持 ELK、Splunk、Webhook 及任意 API 推送。Module 提取 IOC、计算 correlation_uid，同一事件窗口内相关告警归并到同一 Case，**直接减少 80% 以上重复噪音**
2. **AI 自动研判**：Case Analysis Worker 把告警/Artifact/富化信息/历史知识一起塞进 LLM，输出结构化调查报告（判定结论、严重等级、影响面、证据链、攻击时间线、IOC 指标、处置建议）
3. **Playbook 编排**：把 LLM 研判、TI 查询、CMDB 富化、通知、封禁写成 Python 剧本，一键运行。与 SOAR 工作流同构，但可直接调用 AI 能力
4. **Harness Agent 深度集成**：通过 asp-cli 和 Skills 插件，Claude Code、Codex、OpenCode 等 Agent 可直接查 Case、搜日志、查威胁情报，甚至自动写 Module 和 Playbook
5. **多 SIEM 统一入口**：SIEM YAML 声明索引和字段映射，ELK 走 ES|QL，Splunk 走 SPL，查询结果统一进 Case 上下文
6. **威胁情报与 CMDB 富化**：内置 AlienVault OTX、OpenCTI 和 CMDB Provider，IP/Domain/Hash 自动带情报标签和资产身份
7. **知识沉淀**：结案后自动抽取可复用知识，下次相似告警直接召回历史经验
8. **权限与审计**：平台账号 + LDAP，角色 admin/user/viewer，API Key 与 JWT 鉴权，所有操作留痕
9. **低成本二次开发**：新增告警源写一个 Python Module，新增 SIEM 写一个 YAML，Custom 目录热加载，不用重启
10. **开源可私有部署**：MIT，Docker Compose 一键拉起，数据存本地 PostgreSQL 和 RustFS

## 二、技术架构：六层 + 一条数据闭环

技术栈：后端 Django，前端 Vite + Ant Design，CLI 用 Typer/Rich。

**基础设施层**：Docker Compose 单节点——Nginx、Gunicorn、Uvicorn ASGI、5 类 Worker 容器、PostgreSQL 17、Redis Stack、RustFS。

**数据与集成层**：统一 SIEM 查询后端（ELKQueryBackend / SplunkQueryBackend）、威胁情报 Provider、CMDB Provider 和 LLM 路由。LLM 用 LangChain ChatOpenAI 封装，支持按 tag 选择不同模型（结构化输出走支持 json_schema 的模型）。

**智能体运行时（核心）**：BaseModule 负责告警接入，BasePlaybook 负责编排。Custom 目录脚本可覆盖官方脚本，**AST 解析时拒绝相对导入**，保证安全。

**业务编排层**：Redis Stream 做消息队列——Module Worker、Case Analysis Worker、Playbook Worker、ELK Action Worker、Dashboard Cache Worker 各自消费不同 Stream，松耦合且水平扩展方便。

**数据流闭环**：告警输入 → Module 归并 → 生成 Case → AI/剧本调查 → 知识沉淀 → 响应处置。

**实现细节**：
- Redis Stream 消费者组用 `xreadgroup` 无确认模式 + maxlen 削峰，避免消息积压
- LLM 调用统一走 `invoke_structured_llm()`，Pydantic schema 约束输出格式，直接返回可落库字段
- Case Analysis 启动前用 `pg_advisory_xact_lock` 锁 Case，防止重复分析

## 三、告警收敛的关键实现

1. Module 收到告警 → `generate_correlation_uid(rule_id, time_window, timestamp, keys)`：规则 ID、时间分桶、关联键值做 sha256 取前 16 位
2. 同一窗口、同一组键值 → 同一 correlation_uid
3. `create_alert_with_context()` 用 PostgreSQL `pg_advisory_xact_lock` 对 correlation_uid 加事务锁，原子创建 Case/Alert/Artifact/Enrichment——并发安全 + 同一事件只生成一个 Case

**写 Module**：继承 BaseModule，声明 STREAM_NAME 和 THREAD_NUM，重写 run，调用 `create_alert_with_context`。Custom 目录写好脚本前台点"Refresh"即热加载。

**SIEM YAML**：定义 index、time_field、event_id_field、fields 映射——ELK backend 转 ES|QL，Splunk backend 转 SPL。Playbook 里 `self.siem.query(...)` 拿统一格式日志，不关心底层 SIEM。

## 四、安装：两条路径

### 路径 A：Docker Compose 生产部署（最快）

```bash
git clone https://github.com/FunnyWolf/agentic-soc-platform.git
cd deploy/asp-compose
./init.sh          # 生成 .env 和目录结构
./doctor.sh        # 检查端口、依赖、权限
docker compose up -d
./createsuperuser.sh
# 访问 https://<host>/login
```

HTTPS 证书挂 compose volumes；.env 有完整变量表（镜像版本、Django Secret、PostgreSQL 调优）。

### 路径 B：源码开发（适合二次开发）

```bash
cd development/docker && docker compose up -d   # PostgreSQL/Redis/RustFS
cd backend
uv sync
uv run python manage.py migrate
uv run python manage.py runserver                # Web
uv run uvicorn asp.asgi:application              # ASGI WebSocket
cd frontend && pnpm install && pnpm dev
```

Worker 单独起（5 条命令，对应 Module/Case Analysis/Playbook/ELK Action/Dashboard Cache）。自定义脚本放 `backend/custom/` 下 modules/、playbooks/、siem_yamls/ 目录，保存即生效。

## 五、高效使用

**写 Playbook 做自动调查**：继承 BasePlaybook，定义 NAME/DESC/TAGS。run 里 `self.case`/`self.alerts`/`self.artifacts` 读上下文，`self.siem.query()`/`self.threat_intel.lookup()`/`self.cmdb.query()` 做富化，`self.invoke_llm()` 生成结论。中间结果经 `self.data` 字典落地。复杂剧本用 `read_prompt("System")` 加载自定义 Prompt 统一调查风格。

**asp-cli 脚本化**（pipx 安装，配置 ~/.asp/settings.json 自动 0o600 权限）：

```bash
asp-cli case list --limit 5
asp-cli case get --id 42 --output json
asp-cli siem query --siem prod --query "..."
asp-cli playbook run --case-id 123 my_playbook.py
asp-cli ti lookup --value 1.2.3.4
asp-cli dev stream inspect --stream agentic-modules
```

**Harness Agent 直接办案**：装 Skills 后 Agent 获得查询、分析、写代码能力。可直接说："帮我查一下过去 24 小时 Case ID 最大的三个事件，给出研判结论，并写一个 Module 把这类告警自动接入。"

## 六、适用场景

1. 中小安全团队：低成本搭私有 SOC
2. 已有 SIEM/SOAR 想引入 LLM 研判的团队
3. 喜欢自己写脚本、深度定制告警处理逻辑的人

**不是开箱即用的"黑盒"**——需要按自己环境配置告警规则、SIEM 查询、Playbook 步骤。但一旦跑通，调查过程从"翻日志"变成"看 AI 报告 + 做关键决策"。

---

## 战略分析

**ASP 是"Harness 工程 + Agentic 工作流"在安全垂直领域的完整落地**，与知识库已有内容形成强呼应：

| 本文概念 | 知识库已有对应 |
|---------|--------------|
| Harness Agent 集成（Claude Code/Codex/OpenCode） | Harness Engineering 概念页（小米 HarnessX、腾讯大仓）、Agent 治理（Hook 拦截） |
| LLM 结构化输出（invoke_structured_llm + Pydantic） | LangChain 源码解析系列（StructuredPrompt 输入输出合一） |
| 告警收敛（correlation_uid + 事务锁） | 多 Agent 编排（一致性/容错挑战） |
| 知识沉淀（结案自动抽取可复用知识） | 自进化 Agent 调研（经验/Skill 存储型路线） |
| Redis Stream 消息队列五 Worker | 多 Agent 云原生高并发工程落地（银河技术） |

**核心启发：**

1. **"告警收敛"思想可迁移到任何 Agent 事件流**——correlation_uid（规则+时间窗口+关联键 sha256）把海量噪音归并为可调查单元，这与 Hermes 处理微信消息/通知流时的去重归并思路同构。ASP 的 `pg_advisory_xact_lock` 防重复处理是生产级细节。

2. **LLM 结构化输出约束是 Agent 工程的关键模式**——统一 `invoke_structured_llm()` + Pydantic schema 让 LLM 直接返回可落库字段。这与 Hermes 的 function calling / 结构化输出思路一致，ASP 证明了在安全场景的生产可用性。

3. **Harness Agent + Skills 插件模式**是 ASP 最有价值的架构决策——平台能力通过 CLI 暴露给 Claude Code/Codex/OpenCode，Agent 能"直接办案"甚至写 Module/Playbook 自我扩展。这正是 Hermes 技能系统的扩展方向：把平台能力做成 Agent 可调用的工具层。

4. **Custom 目录热加载 + AST 安全校验**（拒绝相对导入）是二次开发的优秀实践——低门槛扩展 + 安全边界并存，与 Hermes"新建独立 skill 不改已有"的偏好精神一致。

5. **对用户的价值**：如果用户做 AI 安全/运维方向（知识库已有"智能 OnCall"归档），ASP 是 Agentic 安全运营的完整参考实现；即使不做安全，其"告警流 → Case → LLM 研判 → 知识沉淀"的流水线设计也适用于任何 Agentic 数据处理系统。

**一句话**：ASP = Agentic 安全运营的完整开源实现，核心价值在"告警收敛 + LLM 结构化研判 + Harness Agent 办案"的流水线设计，二次开发友好（Module/Playbook/SIEM YAML 三件套热加载）。

## 归档日志

- 2026-08-08 归档
