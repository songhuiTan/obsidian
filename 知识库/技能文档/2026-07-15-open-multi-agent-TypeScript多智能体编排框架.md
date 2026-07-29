# [开源] 面向 TypeScript 后端的多智能体编排框架：open-multi-agent

> 来源：一飞开源（微信公众号）
> 日期：2026-07-15
> 链接：https://mp.weixin.qq.com/s/xr6eIBleeX__26WB1hD33w
> GitHub：https://github.com/open-multi-agent/open-multi-agent

---

**目标驱动**的多智能体编排框架。给一个目标，协调器在运行时自动拆解为任务 DAG，并行执行独立任务，综合出带类型、经 schema 校验的结果。

> 让工程师描述目标，而不是流程图。

## 核心范式：目标优先 vs 图优先

| 维度 | 图优先（LangGraph JS） | 目标优先（OMA） |
|------|----------------------|----------------|
| 工作流定义 | 预先枚举每个节点和每条边 | 运行时拆解目标为任务 DAG |
| 适应性 | 为某个目标手工接线 | 计划随目标自适应 |
| 使用方式 | 声明式图编译 | `runTeam(team, goal)` 一次调用 |

与 Anthropic 2026年5月为 Claude Code 推出的 dynamic workflows 是同一押注。

## 技术特性

- **原生 TypeScript**，3 个运行时依赖，可嵌入任意 Node.js 后端
- **MIT 开源协议**
- 提供方：Anthropic、OpenAI 及 OpenAI 兼容端点开箱即用
- Gemini、Bedrock、MCP、Vercel AI SDK bridge 为可选 peer 依赖
- 支持 checkpoint 与恢复：任意 MemoryStore 上对已完成任务做快照，崩溃后用 `restore()` 恢复
- 协调器拆分 → 确定性调度器执行 → 计划始终可审查、可回放

## 快速开始

```bash
npm create oma-app@latest
# 可选模板：PR Review Agent / 安全分析 Agent / multi-agent DAG 入门 Demo
# 可选 provider：云端 / OpenAI 兼容 / 本地 Ollama
```

现有项目集成：

```bash
npm install @open-multi-agent/core
```

## 对比选型

| 如果你需要 | 选择 |
|-----------|------|
| 固定生产拓扑 + 持久化时间旅行生态 | **LangGraph JS** |
| 全栈平台 + 工作流手工接线 | **Mastra** |
| Python 技术栈 + 成熟多智能体生态 | **CrewAI** |
| AI 应用工具包 + 广泛模型支持 | **Vercel AI SDK** |
| **TypeScript 目标到结果自动拆解** | **open-multi-agent** |
