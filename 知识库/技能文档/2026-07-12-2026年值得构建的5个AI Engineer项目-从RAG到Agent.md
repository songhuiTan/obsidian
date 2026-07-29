# 2026 年值得构建的 5 个 AI Engineer 项目：从 RAG 到 Agent 的生产级实践

> 作者：AI研究生
> 来源：微信公众号「AI大模型观察站」
> 日期：2026-07-12 21:05
> 原文：[2026 年值得构建的 5 个 AI Engineer 项目](https://mp.weixin.qq.com/s/iDpaFNXEOZqwJ9PVpMEivQ)

---

## 项目总览

| 项目 | 领域 | 难度 | MVP 估计 | 核心技能 |
|------|------|------|----------|----------|
| **Atlas** | RAG 研究与知识系统 | 中级 | 2-4 周 | RAG/Embeddings/Search Quality/Tool Calling/Evaluation |
| **PrismDoc** | 多模态文档智能 | 中级 | 2-3 周 | Multimodal AI/Structured Extraction/Validation/Human Review |
| **Relay** | 实时语音运营助手 | 中高级 | 2-4 周 | Streaming/WebRTC/Stateful Sessions/Latency Optimization |
| **ForgePilot** | 受控软件工程 Agent | 高级 | 3-5 周 | Orchestration/Sandboxing/Human Approval/Coding Evaluation |
| **DomainLens** | 专用本地模型 + 持续 Evaluation | 高级 | 3-6 周 | Fine-tuning/Quantization/Benchmarking/Model Governance |

---

## 项目 1：Atlas — 证据优先的研究与知识系统

### 核心能力
- 安全 workspace + 文档/网站/数据库连接
- Hybrid keyword + vector retrieval + rerank
- 带引用和冲突来源检测的答案生成
- MCP connectors（已认证 workspace；文档 ingestion；MCP 规范 2025-11-25 确立，企业托管 2026-06-18 稳定）
- Citation Validator：只允许引用 retriever 提供的 source IDs
- Human-in-the-loop 审批

### 技术栈
- 前端：Next.js / React，后端：FastAPI / Django / Node.js
- 数据库：PostgreSQL + pgvector（HNSW indexes），复杂关系用 Neo4j
- 框架：LangGraph（persistence/streaming/HITL）/ OpenAI Agents SDK / LlamaIndex Workflows
- 模型：GPT-5.6 Terra、Claude Sonnet 5、Gemini 3.5 Flash 等

### 关键工程挑战
- 检索质量差 → keyword + vector + rerank
- 幻觉引用 → 只允许引用 retriever 提供的 source IDs
- 文档 prompt injection → 检索到的文本视为不可信数据
- 成本增长 → 小模型做 query classification/caching/summarization

### Evaluation
Retrieval precision/recall、answer correctness、citation accuracy、unsupported-claim rate、tool-call success、cost per task。至少 50 个真实问题的 test set。

---

## 项目 2：PrismDoc — 多模态文档智能平台

### 核心能力
- PDF/Image upload → document classification
- OCR + layout extraction + multimodal LLM
- JSON Schema output + field-level confidence
- Visual bounding boxes + human correction interface
- PII redaction + audit history

### 技术栈
- 前端：React / Next.js，后端：FastAPI
- 队列：Celery / Temporal / Cloud Queue
- 存储：S3 / Cloudflare R2 / Google Cloud Storage
- OCR：Google Document AI / Azure AI Document Intelligence / AWS Textract
- 结构化输出：OpenAI Structured Outputs（Pydantic/Zod 校验）

### 关键工程挑战
- OCR + layout + business validation 分阶段，识别哪个阶段失败
- 用确定性代码重新计算总额，不信任模型算术
- 检测模糊/旋转/缺页后拒绝
- 敏感文档加密、PII 擦除、角色权限、保留规则

---

## 项目 3：Relay — 实时语音运营助手

### 核心能力
- Streaming audio + voice activity detection + interruption
- Tool calling + 不可逆操作前确认
- Live transcript + call summary
- Multilingual support + fallback to text/human

### 技术栈
- 客户端：React / React Native / Flutter
- Transport：LiveKit（WebRTC）+ OpenAI Realtime / Gemini Live
- 模块化方案：Deepgram (STT) + 文本模型 + TTS
- 存储：PostgreSQL + Redis（session）

### 关键工程挑战
- 响应慢 → stream partial audio，减少不必要 model calls
- 误判中断 → 调整 VAD，区分中断与简短回应
- 暴露 narrow tools（如 `reschedule_appointment`），不用不受限的 database/shell

---

## 项目 4：ForgePilot — 受控软件工程 Agent

### 核心能力
- GitHub issue → plan generation → plan approval
- Isolated worktree/container → code modification
- Automated tests + security scanning + review agent
- Retry budget + full execution trace → PR summary

### 技术栈
- Python / TypeScript + Docker + Git worktrees
- LangGraph（checkpoints + human-in-the-loop workflows）
- 确定性步骤：file validation / formatting / test execution / dependency scanning
- 强 coding model（GPT-5.6 Sol / Claude Fable 5）做 planning，快速模型做 summary

### 关键工程挑战
- 无限循环 → 最大步骤数 + token budgets + 明确完成条件
- 破坏性命令 → sandbox + 拒绝危险命令 + 审批
- 代码库中的 prompt injection → comments/issues/docs 视为不可信
- 绝不让 coding agent 标记自己的工作为已接受

### Evaluation
Resolved-issue rate、test pass rate、regression rate、human acceptance、tool failures、steps per task、cost per change。

---

## 项目 5：DomainLens — 带持续 Evaluation 的专用本地模型

### 核心能力
- 领域任务 → 收集清洗代表样本 → training/evaluation dataset → baseline
- Fine-tune open-weight model（LoRA）
- Quantized deployment with vLLM（OpenAI-compatible API）
- Confidence-based fallback to stronger hosted model
- Drift monitoring + human feedback + rollback

### 技术栈
- Open-weight 模型：gpt-oss-20b / gpt-oss-120b（Apache 2.0，2025-08-05）
- 训练：Supervised fine-tuning + preference optimization（LoRA）
- 部署：vLLM + quantization
- Experiment tracking + safety classifier + structured outputs

### 关键工程挑战
- 训练数据差 → 去重 + label definition + 人工检查
- 数据泄漏 → 按 customer/time/source 拆分而不是随机
- 过拟合 → training vs held-out，validation quality 不再提升时停止
- 部署成本 → 测试 quantization/batching/更小模型
- 模型漂移 → 监控 class distribution/confidence/correction rate

### Evaluation
Accuracy、precision、recall、F1、calibration、refusal quality、latency、throughput、memory、cost per 1K tasks、human correction rate。

---

## 推荐学习顺序

1. **PrismDoc** → 学会将 AI 产品拆为 deterministic processing + model reasoning + human review
2. **Atlas** → 学习 embeddings、retrieval、citations、tool use、evaluation
3. **Relay** → 复用 tool design + backend，学习 streaming、state、latency
4. **DomainLens** → 必须有可靠的 evaluation set 才做 fine-tuning
5. **ForgePilot** → 综合 tool use、long-running state、security、testing、model routing、observability

## 常见错误

- 90% 时间花在界面，AI workflow 留作未测试的 prompt
- 固定函数序列能用时却用 Agent
- 不校验模型输出的 JSON schema
- 信任模型算术而不是确定性代码
- 不给敏感数据加密/权限/保留策略
- 说不出 dataset/metric/sample size/testing process 就声称"95% accuracy"
- 在理解 workflow 前安装多个 Agent frameworks

## 三阶段开发路线图

**阶段 1：构建 MVP** — 一个用户、一个问题、一个成功 workflow，不用 multi-agent / 复杂 memory / 过早 scaling

**阶段 2：让它可靠** — evaluation dataset + validation + retries + timeouts + fallback + security + tracing + feedback

**阶段 3：让它生产就绪** — auth + permissions + monitoring + rate limits + data retention + multi-tenancy + cost controls + 文档

---

## 归档信息

- 公众号：AI大模型观察站
- 作者：AI研究生
- 归档日期：2026-07-12
- 原文链接：https://mp.weixin.qq.com/s/iDpaFNXEOZqwJ9PVpMEivQ
