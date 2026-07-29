# Bifrost：开源 AI 网关，比 LiteLLM 快 50 倍

> 来源：深度记事 (codexlz) · 2026-06-10
> 原文：https://mp.weixin.qq.com/s/W-3sCUqQ-pLA5DUtidS7mg
> 分类：工具与实用技能

---

> Bifrost 是 AI 流量的网关层，放在应用和模型供应商之间，统一 OpenAI 兼容 API + 路由/故障切换/语义缓存/虚拟密钥/可观测性/MCP 工具治理。GitHub: https://github.com/maximhq/bifrost

---

## 核心定位

AI Gateway，在多个模型供应商前面提供统一 OpenAI 兼容 API，集中处理：

- 路由与故障切换（fallback + 加权负载均衡）
- 语义缓存
- 虚拟密钥管理
- 可观测性（Prometheus 指标 + OpenTelemetry）
- 治理控制
- MCP 工具治理

设计思路：你的应用和 Bifrost 对话，Bifrost 再去和模型供应商对话。路由和控制逻辑放在网关里，而不是塞进业务代码。

---

## 性能对比（厂商自测）

AWS `t3.medium`，500 RPS 测试：

| 指标 | Bifrost | LiteLLM |
|------|---------|---------|
| 成功率 | 100% | 88.78% |
| P50 延迟 | 804ms | 38.65s |
| P99 延迟 | 1.68s | 90.72s |
| 最大延迟 | 6.13s | 92.67s |
| 吞吐量 | 424 req/s | 44.84 req/s |
| 峰值内存 | 120MB | 372MB |

在 5,000 req/s 持续压测下，Bifrost 每个请求只增加 11 微秒额外开销。

---

## 快速启动

```bash
npx -y @maximhq/bifrost
```

默认在 `localhost:8080` 启动 HTTP 网关。Web UI 提供供应商配置、请求日志、指标、分析、虚拟密钥和治理控制。

```bash
curl -X POST http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "openai/gpt-4o-mini",
    "messages": [{"role": "user", "content": "Hello, Bifrost!"}]
  }'
```

也支持 Docker 部署。

---

## MCP 支持

Bifrost 同时提供 MCP Gateway 侧：
- 既能当 MCP client，也能当 MCP server
- 连接外部工具服务器，也把工具暴露给 Claude Desktop 等客户端
- 支持工具过滤、OAuth、工具执行控制和 Agent mode

说明 Bifrost 想做的不只是模型路由，还包括面向工具的 Agent 基础设施。

---

## 适用场景

- 多团队共享的内部 AI 平台
- 带模型能力的 SaaS 产品
- 需要日志、控制和私有化部署路径的企业环境
- 既要模型路由、又要工具治理的 Agent 系统

---

## 关键认知

Bifrost 真正有价值的不是"统一 API"本身，而是围绕这个 API 的操作层：路由、故障切换、可观测性、使用控制、缓存和治理。随着 AI 系统横跨多个团队、多个产品和多个场景，这些东西往往比"能不能调模型"更重要。

**官网**：https://www.getmaxim.ai/bifrost
**GitHub**：https://github.com/maximhq/bifrost
**Benchmark**：https://www.getmaxim.ai/bifrost/resources/benchmarks
