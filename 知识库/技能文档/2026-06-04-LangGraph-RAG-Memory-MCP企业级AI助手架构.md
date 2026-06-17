---
title: LangGraph + RAG + Memory + MCP 企业级 AI 助手架构
author: 技术自由圈
source: "https://mp.weixin.qq.com/s/IxMH4KZbS63suk18bjHTRg"
date: 2026-06-04
tags:
  - AI架构
  - LangGraph
  - RAG
  - MCP
  - Agent
  - 企业级架构
---

# LangGraph + RAG + Memory + MCP 企业级 AI 助手架构

本文基于六大架构思维，以 LangGraph 状态编排框架为核心，深度融合分层记忆系统、工业级 RAG 流水线、MCP 标准化能力接入体系，构建一套完整、可落地的企业级 AI 助手架构方案。

**四层架构概览：**
- 接入层多协议承接流量，安全层完成鉴权与风控
- LangGraph 编排层依托中心化 State 管控全链路状态与分支流程
- 能力层集成六阶流水线 RAG、三层分级记忆、MCP 标准化工具与合规引擎
- 存储层依托 Milvus、PostgreSQL、Redis 分层存数，观测层全链路监控埋点

**完整链路：** 请求校验 -> 记忆召回 -> 查询优化 -> 路由分发 -> RAG 检索 -> 工具调用 -> 内容审核与应答输出

![四层架构总览](assets/2026-06-04-langgraph-rag-memory-mcp/img_001.jpg)

> 注：本文为单体架构。下一版本将演进为 A2A 分布式联邦多 Agent 架构。

---

## AI 架构设计六大思维

生产级 AI 系统的核心竞争力不在于单模块技术深度，而在于架构设计的合理性与前瞻性。

![六大架构思维](assets/2026-06-04-langgraph-rag-memory-mcp/img_002.jpg)

### 1. 状态中心化思维：一切流转皆可追溯、可恢复

摒弃传统无状态 API 开发思维，以 LangGraph State 为系统唯一数据流转载体，所有节点交互、工具调用、记忆检索、RAG 结果均统一归集至中心状态。通过标准化状态规约与持久化机制，实现服务重启、节点中断、流量波动场景下的**状态无缝恢复、流程断点续跑**。

### 2. 关注点分离思维：高内聚、低耦合的分层解耦

严格拆分流量接入、安全治理、流程编排、能力实现、数据存储、可观测六大层级，每层职责单一、边界清晰。RAG、记忆、工具等核心能力以插件化方式接入编排层，支持独立迭代、单独扩容、按需启停。

### 3. 能力标准化思维：统一协议、统一规范、统一治理

基于 MCP 协议统一所有内外能力接入标准，实现本地函数、远程服务、数据库、知识库的接口归一化。统一错误处理、超时熔断、权限校验、日志输出规范。

### 4. 分层容错思维：多级降级、故障隔离、风险可控

构建"接口层-编排层-能力层-数据层"四级容错体系，配置差异化降级策略。通过熔断、重试、兜底、人工介入机制，保障极端场景下服务不中断。

### 5. 成本性能平衡思维：精细化 Token 与算力管控

通过上下文压缩、记忆分层、RAG 重排序、动态截断、模型分级调用等策略，精准平衡**回答质量、Token 消耗、响应延迟**三者关系。

### 6. 全链路可观测思维：可追踪、可量化、可优化

将观测体系贯穿系统所有层级与核心流程，基于量化数据持续迭代优化，形成"观测-分析-优化-迭代"的闭环。

---

## 一、架构总览：六层生产级弹性技术栈

传统四层架构（接入层、能力层、存储层、应用层）仅适用于原型验证。本文基于六大架构思维，迭代升级为**六层生产级弹性技术栈**。

![六层技术栈](assets/2026-06-04-langgraph-rag-memory-mcp/img_003.jpg)

### 1. 用户接口层：多协议统一接入

- RESTful API 同步接口
- SSE 流式输出
- WebSocket 长连接实时交互
- RabbitMQ/Kafka 消息队列异步处理

### 2. 流量治理与安全层

- OAuth2.0/JWT 身份认证（AuthN）、RBAC 权限授权（AuthZ）
- 接口速率限制与并发管控
- 恶意请求拦截、输入参数清洗与脱敏
- SQL/提示词注入防护
- 严格约束 thread_id、user_id、tenant_id 等隔离参数仅由服务端生成

### 3. LangGraph 编排层：状态驱动的智能流程中枢

基于**有向状态图（StateGraph）**实现声明式、可循环、可分支、可回滚的复杂业务流程编排。通过原子化节点封装单一能力、条件边实现动态路由、检查点机制实现状态持久化与故障续跑。

### 4. 能力模块层：插件化标准化能力仓库

四大核心模块：
- **工业级 RAG 流水线：** 查询预处理、语义重写、多源混合检索、交叉编码器重排序、上下文智能压缩、答案溯源校验
- **分层记忆管理器：** 短期会话记忆、长期情节记忆、结构化事实记忆的分层存储与智能更新
- **MCP 统一工具执行器：** 基于模型上下文协议归一化封装本地函数与远程服务
- **合规与策略引擎：** 业务规则、成本管控、内容审核、敏感操作拦截

### 5. 数据与基础设施层

- Milvus 向量数据库（向量化知识与长期记忆）
- PostgreSQL（结构化数据、会话配置、Graph 检查点、审计日志）
- Redis 缓存（活跃会话状态、高频查询结果）
- 对象存储（原始文档、大文件资源）

### 6. 全链路可观测层

深度融合 LangSmith、OpenTelemetry、Prometheus+Grafana、结构化日志，实现请求全链路追踪、核心指标监控、日志结构化检索。

---

## 二、状态系统设计：LangGraph State 生产级规范

状态是 LangGraph 工作流的核心载体。原型项目中粗放的状态设计会直接导致后期系统迭代困难、数据混乱、会话异常。

![状态系统设计](assets/2026-06-04-langgraph-rag-memory-mcp/img_004.jpg)

### 1. 核心设计原则

- **最小完备性原则：** State 仅保留跨节点共享的核心数据，单会话状态字段严格精简
- **数据归约一致性原则：** 使用框架内置归约器实现数据追加而非覆盖
- **会话强隔离原则：** 以服务端生成的 thread_id 为唯一会话隔离标识
- **可序列化原则：** 所有状态字段均采用可序列化数据结构

### 2. 生产级 State 定义

```python
from typing import Annotated, List, Optional, Literal, Dict, Any
from typing_extensions import TypedDict
from langgraph.graph.message import add_messages

class GraphState(TypedDict):
    """生产级 LangGraph 中心状态：标准化、可持久化、可追溯"""
    # 多轮对话消息历史，归约器追加更新
    messages: Annotated[list, add_messages]
    # 用户原始输入与预处理后的标准化查询
    human_input: str
    refined_query: Optional[str]
    # 工作流路由决策字段
    next_node: Optional[Literal["retrieve_memory", "retrieve_rag", "call_tool", "direct_answer"]]
    # RAG 检索结果与重排序后优质文档
    raw_retrieved_docs: List[dict]
    ranked_retrieved_docs: List[dict]
    # 分层记忆检索结果
    relevant_short_memory: List[dict]
    relevant_long_memory: List[dict]
    relevant_struct_memory: List[dict]
    # 工具调用相关数据
    tool_call_list: List[dict]
    tool_exec_results: List[dict]
    tool_error_info: Optional[str]
    # 人工审核与合规管控
    needs_human_approval: bool
    sensitive_check_result: str
    # 溯源与可观测字段
    node_execute_logs: List[dict]
    token_consumption: Dict[str, int]
    # 服务端可信配置
    runtime_config: Dict[str, Any]
```

### 3. 生产级持久化与故障恢复

开发环境默认的内存级检查点（AsyncSqliteSaver）无法适配生产场景。需采用**分布式持久化检查点方案**：
- PostgreSQL Saver 实现高可靠状态持久化
- Redis 实现热点会话状态缓存
- 配置状态快照定时备份、过期会话自动清理、异常状态回滚
- 支持节点执行失败、服务重启、流量熔断场景下的**精准断点续跑**

---

## 三、分层记忆系统：企业级持久化智能记忆

生产级记忆系统需模拟人类"瞬时记忆-短期记忆-长期记忆"的分层机制。

![分层记忆系统](assets/2026-06-04-langgraph-rag-memory-mcp/img_005.jpg)

### 1. 三层记忆核心能力

#### L1 短期工作记忆（会话级）
依托 LangGraph Checkpointer 与 GraphState 消息列表，生命周期绑定当前会话 thread_id。采用**滑动窗口截断策略**保留最近 8-12 轮核心对话。数据存储于 Redis 热层。

#### L2 结构化事实记忆（用户画像级）
从对话中自动提取结构化键值对（身份、偏好、权限、常用操作、任务记录等），存储于 PostgreSQL。支持精准匹配查询，支持记忆更新、修正、删除的自动与人工双机制。

#### L3 长期情节记忆（跨会话级）
对全量历史对话进行 LLM 摘要压缩，过滤冗余话术，提炼核心业务意图与对话结论，向量化后存储于向量数据库。支持跨天、跨会话的智能延续。

### 2. 企业级三级存储分层架构

| 层级 | 存储介质 | 用途 | 特性 |
|------|---------|------|------|
| L1 热层 | Redis | 活跃会话上下文、临时状态 | 亚毫秒级响应，过期自动销毁 |
| L2 温层 | PostgreSQL | 用户画像、会话元数据、审计日志 | 事务一致性，复杂条件查询 |
| L3 冷层 | 向量数据库 | 长期对话记忆、知识库向量 | 语义检索，海量数据沉淀 |

### 3. 生产优化策略

- **记忆遗忘：** 自动淘汰低频、无效、过期记忆
- **记忆合并：** 合并重复相似记忆片段
- **优先级排序：** 根据交互频次、业务重要性分级，优先加载高价值记忆

---

## 四、工业级 RAG 流水线

传统简易 RAG 存在检索精准度低、上下文冗余、LLM 优先依赖自身知识编造答案、无溯源能力等问题。

![RAG 流水线](assets/2026-06-04-langgraph-rag-memory-mcp/img_006.jpg)

### 1. 六阶全链路优化流程

#### (1) 查询预处理与语义重写
通过轻量 LLM 完成语义补全、歧义消除、意图识别、查询扩展。

#### (2) 多源混合检索
融合稠密向量检索（语义理解）与 BM25 稀疏检索（关键词精准匹配），通过加权融合算法合并两路结果。

#### (3) 交叉编码器重排序
对 Top20 初筛结果精排，仅保留 Top5 核心优质片段。

#### (4) 上下文智能压缩
精简冗余语句、无效格式、重复内容，保留核心有效信息。

#### (5) 强制溯源提示工程
加入**强制引用、禁止编造、明确兜底、来源标注**四大约束规则，为每段检索文档添加唯一来源标识。

#### (6) 答案校验与脱敏
反向校验内容是否完全匹配检索文档，自动过滤敏感信息。

### 2. 核心代码实现

```python
from langchain.retrievers import BM25Retriever, EnsembleRetriever
from langchain_community.vectorstores import Qdrant
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.rerankers import CrossEncoderReranker

# 1. 初始化生产级检索组件
embeddings = OpenAIEmbeddings(model="text-embedding-3-large")

vector_store = Qdrant(
    url="your-qdrant-cluster-url",
    collection_name="enterprise-docs",
    embedding_function=embeddings
)
vector_retriever = vector_store.as_retriever(search_kwargs={"k": 20})

bm25_retriever = BM25Retriever.from_existing_index("enterprise-doc-index")

# 混合检索加权融合
ensemble_retriever = EnsembleRetriever(
    retrievers=[vector_retriever, bm25_retriever],
    weights=[0.7, 0.3]
)

reranker = CrossEncoderReranker(model="cross-encoder/ms-marco-MiniLM-L-6-v2")

# 2. 标准化检索节点
async def rag_retrieve_node(state: GraphState) -> dict:
    query = state.get("refined_query") or state.get("human_input")
    raw_docs = await ensemble_retriever.ainvoke(query)
    ranked_docs = reranker.rerank(query, raw_docs)[:5]
    formatted_docs = [f"[来源{i+1}] {doc.page_content}" for i, doc in enumerate(ranked_docs)]
    return {
        "raw_retrieved_docs": raw_docs,
        "ranked_retrieved_docs": ranked_docs,
        "retrieved_docs": formatted_docs
    }

# 3. 生产级增强提示词构建
def build_production_prompt(state: GraphState) -> str:
    base_prompt = "你是企业级专业AI助手，回答必须精准、严谨、合规。\n\n"
    if state.get("relevant_long_memory") or state.get("relevant_struct_memory"):
        base_prompt += f"\n【用户历史背景信息】{state['relevant_long_memory'] + state['relevant_struct_memory']}"
    if state.get("retrieved_docs"):
        base_prompt += f"""
【权威参考文档】
{"\n".join(state['retrieved_docs'])}
【强制回答规则】
(1) 所有回答必须严格基于上述参考文档与用户历史信息，禁止编造
(2) 无匹配信息时，必须明确回复"根据已知信息无法回答该问题"
(3) 回答需简洁精准，优先引用参考文档核心内容
(4) 禁止输出与问题无关的冗余内容，严格规避信息幻觉
        """
    return base_prompt
```

---

## 五、MCP 协议集成：标准化能力接入

MCP 核心解决传统工具集成"N对N耦合、适配成本高、无法动态发现、安全边界模糊"的痛点。

![MCP 协议集成](assets/2026-06-04-langgraph-rag-memory-mcp/img_007.jpg)

### MCP 核心生产价值

- **彻底解耦：** 工具服务端与客户端完全解耦，一次实现 MCP 标准化接口即可被所有 Agent 调用
- **动态发现：** Agent 启动时自动扫描可用工具，获取名称、参数 Schema、功能描述
- **安全隔离：** MCP 服务独立部署，与核心 Agent 进程物理隔离，形成安全沙箱
- **统一治理：** 所有工具调用统一接入、监控、容错、审计

### 本地工具 vs MCP 远程工具

| 类型 | 部署方式 | 适用场景 |
|------|---------|---------|
| 本地工具 | @tool 装饰器，运行于 Agent 进程内 | 高频、轻量、无外部依赖的计算/格式化/校验 |
| MCP 远程工具 | 独立进程/服务部署 | 重型计算、网络请求、数据库操作、敏感业务 |

---

## 六、LangGraph 高级编排：生产级工作流

基于状态设计、记忆系统、RAG 流水线、MCP 工具体系，通过 LangGraph 声明式状态图编织为**可循环、可分支、可审核、可恢复**的完整智能工作流。

![LangGraph 编排](assets/2026-06-04-langgraph-rag-memory-mcp/img_008.jpg)

### 核心编排节点

- 记忆检索节点
- 查询预处理节点
- 路由决策节点
- RAG 检索节点
- Agent 推理节点
- 工具执行节点
- 合规审核节点
- 结果输出节点

---

## 七、生产级落地：部署、容错、观测与治理

### 1. 高可用部署架构

Docker 容器化 + K8s 集群编排，数据库/向量库/Redis 均采用集群部署，配置主从备份、定时快照、异地容灾。

### 2. 多级容错与降级体系

- 检索失败 -> 自动回落关键词匹配
- 工具调用超时 -> 触发熔断
- LLM 异常 -> 返回标准化兜底应答
- 配置重试机制与退避策略

### 3. 全链路可观测体系

- **链路追踪：** LangSmith + OpenTelemetry 可视化节点执行路径
- **指标监控：** Prometheus + Grafana（QPS、P95/P99延迟、错误率、召回率、Token 消耗）
- **结构化日志：** structlog 输出标准化 JSON 日志，敏感数据自动脱敏

### 4. 安全与合规治理

- 输入输出双向内容安全过滤
- RBAC 权限模型管控工具、知识库、记忆访问
- 全流程操作留痕，支持合规审计
- 精细化 Token 统计与成本分摊

---

## 八、前瞻：跨服务分布式 A2A 联邦架构

单体架构无法解决多业务场景、多独立 Agent 的复杂调度问题。

![A2A 联邦架构](assets/2026-06-04-langgraph-rag-memory-mcp/img_009.jpg)

多业务场景 = 每个场景一个独立子图/独立 Agent：
- 财务 Agent（自有 tools、prompt、流程）
- 人事 Agent
- 售后 Agent（自有 RAG）

**顶层只做路由分发 -> A2A 调用**，结构清晰、隔离干净、可独立部署迭代。

主调度 Agent 负责场景识别与路由分发，通过标准化 A2A 接口调用各独立业务服务。

---

## 总结

本架构核心价值：
1. **状态中心化管控：** 解决多节点流转数据丢失、上下文断裂问题
2. **分层智能记忆：** 实现真正的个性化、跨会话连续智能
3. **MCP 标准化能力生态：** 解决工具与数据源碎片化接入，实现能力可插拔、可扩展

全链路可观测、多级容错、安全治理体系为系统规模化稳定运行提供工程化兜底。下一版本可演进为 A2A 联邦架构：主调度 Agent + 独立 Agent 工作者协作模式。

---

> **来源：** 微信公众号"技术自由圈"
> **归档日期：** 2026-06-04
