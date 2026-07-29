---
source: "45岁老架构师尼恩 / 技术自由圈"
url: "https://mp.weixin.qq.com/s/bjMy_oK-h5zHgDxGUwQSjw"
date: 2026-07-14
tags: Langfuse, RAGAS, 可观测, 评测, 生产级基建, LLMOps
---

# 滴滴面试：如何设计一个 Langfuse+RAGAS 可观测+评测 生产级基建

> 基于 Langfuse+RAGAS 的 LLM 全链路可观测与评测一体化方案，解决 RAG/Agent 黑盒问题。

## 背景：从"肉眼判断"到数据驱动

传统 RAG/Agent 优化是**肉眼判断模式**——人工拉取几十条对话，凭主观感受判定好坏，存在严重问题：

- 样本量极小、主观偏差大
- 无法量化——幻觉 28%→12% 这种数据差异统计不出来
- 无法定位——是检索、改写还是 Prompt 的问题？
- 项目复盘拿不出客观数据

**Langfuse 解决"看得见问题"，RAGAS 解决"评得出好坏"**。

## Langfuse 核心：全链路埋点可观测

### 项目定位

德国团队开源的 **LLM 原生全链路可观测与评测一体化平台**，基于 OpenTelemetry (OTel) 构建。核心优势：开源无厂商锁定、全量私有化部署、适配内网合规。

提供 SaaS 云端版和 Docker Compose 自托管版（PostgreSQL + Redis + ClickHouse + MinIO）。

### 三层核心概念

| 概念 | 层级定位 | 业务含义 |
|------|----------|----------|
| Trace | 顶层根节点 | 一次完整用户请求，整条链路的根容器 |
| Span | 中间子节点 | Trace 内拆分的任意业务步骤（RAG检索/摘要/工具调用） |
| Generation | 叶子终端节点 | 直接调用 LLM API 的最小单元，强制记录 Prompt/模型/Token/耗时/异常 |

### @observe 装饰器

SDK v3 关键语义变化：`@observe()` 装饰出来的是 root Observation（Span/Generation），Trace 由 OTel 自动创建。要挂 `user_id`/`session_id` 需走 `get_client().update_current_trace()`。

三种用法：

| 写法 | 数据类型 | 场景 |
|------|----------|------|
| `@observe(name="链路名")` | Trace | 最外层入口 |
| `@observe()` | Span | 中间业务逻辑 |
| `@observe(as_type="generation")` | Generation | LLM 调用 |

嵌套调用自动生成树形链路，无需手动传递 Trace ID。

### OTel 上下文注入（生产方案）

通过 `opentelemetry.trace.get_current_span().set_attribute()` 注入业务属性：

- `langfuse.user.id` — 用户标识，后台筛选单用户全部历史
- `langfuse.session.id` — 会话标识，串联多轮对话
- `langfuse.tags` — 标签数组（prod/dev/test/v1.2）
- `metadata` — 自定义元数据（设备号、渠道、IP）

**关键原理**：OTel 在 Python 中用 `contextvars` 隐式传递上下文（类似 ThreadLocal，但支持 asyncio），`@observe` 在装饰器入口/出口自动 `attach/detach`，子函数无需显式传参即可继承父 Span 上下文。

**跨进程传递**：通过 W3C Trace Context 标准 HTTP Header（`traceparent`）传递，Propagator 自动 inject/extract。

**坑**：新线程/ThreadPoolExecutor/asyncio.create_task 中 ContextVar 不会自动带过去，需要显式 `context.copy()` 再 attach。

## LangGraph 集成

两层埋点原则：
1. 外层 `graph.invoke()` 入口用 `@observe(as_type="chain")` 注入用户/会话上下文
2. Graph 内每个 Node 函数加 `@observe()` 作为 Span

上下文自动透传，LangGraph 单轮 invoke 全程共享同一 OTel 上下文。条件分支、循环重试、多工具并行调用均可自动生成分支 Span。

## 数据上报 Flush 机制

| 场景 | 策略 |
|------|------|
| 一次性脚本 | 末尾手动 `get_client().flush()` |
| 交互式多轮 | 每轮结束 flush |
| Web 服务 | 请求生命周期结束钩子中 flush |
| 常驻服务 | SDK 后台线程自动定时刷盘，优雅退出自动 flush |

进程退出有 atexit 钩子兜底，仅 kill -9 可能会丢失缓冲区数据。

## RAGAS 量化评测

### 四大核心指标

| 指标 | 卡住环节 | 说明 |
|------|----------|------|
| Context Recall | Retriever | 检索是否包含回答问题所需全部关键信息 |
| Context Precision | Retriever | 召回文档中有效信息占比 |
| Faithfulness | Generator | 输出是否完全来自检索上下文（幻觉检测） |
| Answer Relevancy | Generator | 回复是否精准对应用户提问 |

### 完整闭环流程

1. **线上埋点采集**：Langfuse 实时记录全链路数据，自动捕获异常 Bad Case
2. **人工沉淀数据集**：Trace 详情页 → 打分 → 一键加入 Dataset，打造专属评测题库
3. **自动化量化跑分**：RAGAS 批量评测，对比 Prompt/Chunk 策略/模型版本优劣
4. **分数回写溯源**：评测结果通过 `client.score(trace_id=..., name="ragas/faithfulness", value=...)` 绑定原 Trace

### 注意事项

- Dataset 中 `expected_output` 默认是模型实际回答，若要跑 Context Recall/Precision 需手动替换为 ground truth
- 无 ground truth 时，Faithfulness 和 Answer Relevancy 仍可用（无需标准答案）

## RAG 全链路埋点分层

Trace 根节点 → Span1: Query 改写 → Span2: 向量检索 → Span3: 文档重排 → Generation: LLM 生成

| 阶段 | 必须记录的字段 |
|------|----------------|
| 根 Trace | `user_id`, `session_id`, `tags`, `metadata` |
| Query 改写 | `original_query`, `rewritten_query` |
| 向量检索 | `search_query`, `retrieved_docs`, `scores` |
| 重排 | `final_contexts` |
| Generation | `answer`, `contexts`, `prompt`, `token usage` |

## 常见排坑

1. **云端 SaaS 上报超时**：添加日志屏蔽 + 配置 HTTP 代理，或切 Docker Compose 私有化部署
2. **进程结束后看不到数据**：末尾必须执行 `get_client().flush()`
3. **Generation 无法自动抓取 Token**：部分第三方 LLM SDK 无法自动解析 usage，需手动传入
4. **嵌套调用链路层级混乱**：确保所有子函数全部添加 `@observe`，不能有未埋点的中间函数跳转
