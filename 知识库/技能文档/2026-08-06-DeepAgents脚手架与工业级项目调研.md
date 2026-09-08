# DeepAgents 脚手架生态与工业级项目调研

> 归档：2026-08-06
> 来源：gbrain 知识库检索 + GitHub 开源项目调研（多轮对话汇总）
> 标签：DeepAgents, LangChain, Skill, 脚手架, 工业级, Agent框架

---

## 调研背景

围绕三个问题展开：① DeepAgents 有没有脚手架项目支持开发自己的 Agent；② DeepAgents 与 LangChain 差异；③ DeepAgents 是否天然支持 Skill；④ 有无工业级项目实现可参考。

---

## 一、DeepAgents vs LangChain：不是竞争，是上下层关系

| 维度 | LangChain | DeepAgents |
|------|-----------|------------|
| 本质 | **底层开发库**（LLM 调用抽象、Prompt 模板、Tool 定义、LCEL 链式组合） | **高层 Agent 框架**，构建在 LangChain/LangGraph 之上 |
| 定位 | 工具箱：给零件，自己组装 | 成品骨架：`create_deep_agent()` 一行出 Agent |
| 关系 | 底座 | `LangChain → LangGraph（图引擎）→ create_agent（ReAct 循环）→ create_deep_agent（工业级能力）` |

**核心结论：DeepAgents 不是新框架，是 LangChain 之上的「工业级默认配置」。** LangGraph 里要手写的文件读写、上下文摘要、子任务委派、HITL——DeepAgents 默认全带（17 个内置中间件 + VFS + SubAgent）。

## 二、Skill 支持：一等公民

- 架构上明确：LangGraph「没有 Skill 抽象」→ DeepAgents「**按需加载的可复用行为包**」
- `deepagents init` 生成的布局自带 `src/skills/` 目录（自动 seed 进运行时）
- `deepagents skills create` 子命令直接脚手架新 Skill
- 环境变量 `DEEPAGENTS_SKILLS_PATHS` 指定外部 Skill 路径（多路径冒号分隔）
- 官方内置 skill-creator（从 anthropics/skills 移植），带 `init_skill.py` + `quick_validate.py`
- 运行方式：Agent 按需加载，不占用基础上下文——与 Hermes skill 系统同构

## 三、脚手架方案（三档）

### 1. 官方 CLI：`deepagents init`（最直接）

```bash
uv add "deepagents[cli]"   # 或 pip install "deepagents[cli]"
deepagents init            # 脚手架 deepagents.toml + src/ 结构
deepagents dev             # 本地运行
deepagents deploy          # 部署到 LangSmith
```

生成的布局：
```
src/
  AGENTS.md        # 必需 — system prompt
  skills/          # 可选 — 自动 seed 到 /skills/
  mcp.json         # 可选 — HTTP/SSE MCP servers
  deepagents.toml  # agent 配置：name/model/sandbox
```

`deepagents.toml` 示例：
```toml
[agent]
name = "my-agent"
model = "anthropic:claude-sonnet-4-6"

[sandbox]  # 可选，省略则工具进程内运行
provider = "langsmith"   # none | langsmith | daytona | modal | runloop
scope    = "thread"
```

### 2. 官方模板仓库：`langchain-ai/deep-agent-template`（部署级）

- 可部署的 deep agent graph：`src/deep_agent/graph.py`
- 显式工作流 prompt（plan, delegate, critique, finalize）
- **researcher + critic 两个预置子代理**
- execute/write_file 的 HITL 中断
- uv 管理 + Makefile + starter tests
- 快速开始：`uv sync && cp .env.example .env && uv run langgraph dev`

### 3. 第三方增强：`vstorm-co/full-stack-ai-agent-template`

- cookiecutter 模板，`--ai-framework deepagents` 选项
- 内置文件操作/代码执行/任务管理工具（ls, read_file, write_file, edit_file, glob, grep, execute, write_todos, task）
- `DeepAgentsAssistant` 类带 run()/stream()
- **WebSocket + HITL 审批流**（`DEEPAGENTS_INTERRUPT_TOOLS` 配置哪些工具需 approve/edit/reject）

## 四、工业级项目实现（GitHub）

| 项目 | 定位 | 亮点 |
|------|------|------|
| `emanueleielo/deepagents-open-lovable` | ⭐ 最接近工业级：开源版 Lovable | 自然语言生成完整 React 应用；子代理架构（设计/图像研究专用代理）；实时预览；一键部署 Vercel；完整产品形态 |
| `copilotkit/opengenerativeui` | 生成式 UI 平台 | Agent 端 `create_deep_agent()` + LangGraph；`langgraph.json` 部署到 LangSmith 云（生产托管+追踪+持久化）；前端+Agent 双端完整 |
| `hzhaoy/deep-code-agent` | 编程助手 | 代码审查/测试生成/文档/调试/重构五个专用子代理 + HITL 审批流 + 文件系统集成；最像"Claude Code 的 DeepAgents 复刻" |
| `bodanlabs/deepagents-showcase` | 全栈展示 | FastAPI 跑 Deep Agent + React 前端（assistant-ui），SSE 流式；uv + pnpm + Docker Compose，技术栈最干净，适合当架构样板 |
| `denstis/deep_agents_gui` | 桌面多 Agent | CustomTkinter GUI；10+ 工具按风险分级管理 |

## 五、建议路径

1. `deepagents init` 跑通最小结构
2. 参考 deep-agent-template 加子代理和 HITL
3. 需要前端再上 vstorm 模板
4. 精读 deepagents-open-lovable（Agent 架构设计）+ opengenerativeui（LangSmith 生产部署链路）

## 六、踩坑提示（来自 GitHub issues）

- `create_deep_agent` 默认无条件构造 FilesystemMiddleware（主 agent + 通用子代理 + 每个 inline 子代理），eviction 参数（tool_token_limit_before_evict 20000 / human_message_token_limit_before_evict 50000）硬默认，无 passthrough——自己加 FilesystemMiddleware 会撞 duplicate-middleware 断言
- Skills 在 FilesystemBackend 下正常，但 Daytona 沙箱下曾出现找不到的问题（#1862，已修）
- `create_deep_agent(skills=[...])` 直接实例化 SkillsMiddleware，无 prompt 定制 passthrough（#3137）
- 模型必须支持 tool calling；额外 SDK 依赖（langchain-openai 等）；默认带的工具不少，第一次跑先看默认装了啥

## 归档日志

- 2026-08-06 归档（gbrain 检索 + GitHub 调研）
