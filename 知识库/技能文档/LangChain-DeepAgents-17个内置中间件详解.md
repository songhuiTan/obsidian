---
source: 微信公众号
author: AnthroTech AI
account: AnthroTech AI
date: 2026-06-28 13:30
url: https://mp.weixin.qq.com/s/Tvzz0tX4JAYM-TYYj2MkTQ
tags: LangChain, Deep Agents, Middleware, Agent工程, 上下文管理, 安全审计, 模型路由, 子代理
---

# 深入解析 LangChain / Deep Agents：17 个开箱即用的中间件

> LangChain / Deep Agents 中，Middleware 是一种可插拔的拦截器，包裹在 Agent 的模型调用、工具调用、消息流转等关键环节之外，无需修改核心逻辑就能扩展行为。通过 `create_agent(model, tools, middleware=[...])` 传入即可。

---

## 一、上下文管理类（4个）

### 1. SummarizationMiddleware（历史摘要）
token 接近上限时，自动把较早的消息压缩成摘要，保留最近若干条原样。

```python
SummarizationMiddleware(
    model="gpt-5.4-mini",     # 用便宜模型生成摘要
    trigger=("tokens", 4000), # 达到 4000 token 触发
    keep=("messages", 20),   # 保留最近 20 条消息
)
```

**trigger 支持 AND/OR 组合：**
```python
trigger=[
    {"tokens": 5000, "messages": 3},  # AND
    {"tokens": 3000, "messages": 6},  # AND
],  # 两条之间是 OR
```

⚠️ 摘要是文本级压缩，被摘要的旧消息只保留文字。图片/音频为主的场景建议媒体存对象存储，只传 URL。

### 2. ContextEditingMiddleware（上下文编辑）
与 Summarization 类似但策略更「硬」——直接清除较早的工具调用结果，不依赖额外 LLM 调用。

```python
ContextEditingMiddleware(
    edits=[ClearToolUsesEdit(keep_count=3)]  # 只保留最近 3 次工具调用结果
)
```

### 3. FilesystemMiddleware（文件系统与持久记忆）
给 Agent 注入持久化文件系统和长期记忆能力。

```python
FilesystemMiddleware(
    composite_backend=CompositeBackend(
        workspaces={"/code": None, "/docs": None},
        backend=StoreBackend(FilePathStore("/data/store")),
        routes={"/memories/*": "store"},   # 路由到持久化存储
    )
)
```

### 4. TodoListMiddleware（待办清单与进度管理）
为 Agent 注入 todo/checklist/implement 三个工具，让 Agent 规划、跟踪、汇报进度。

```python
TodoListMiddleware(
    max_todos=10,              # 最多 10 条待办（默认 20）
    default_model="gpt-5.4-mini",  # 用便宜模型做规划
)
```

支持 `finalize_after_completion`：所有待办完成后自动输出总结报告。

---

## 二、安全与合规类（3个）

### 5. PIIMiddleware（PII 脱敏）
在 Agent 调用链中拦截可能泄露的敏感信息。

```python
PIIMiddleware(
    pii_types=[           # 要检测的 PII 类型
        "email", "phone_number", "credit_card", "api_key",
        "ip_address", "person_name"
    ],
    mode="redact",        # 'redact' 替换为占位符；'block' 拦截；'audit' 仅记录
    llm_scan_mode=None,   # 用 LLM 补充检测
)
```

### 6. ToolRetryMiddleware（工具调用自动重试）
工具调用失败时自动重试，可配置重试策略（固定/指数退避/自定义时间）。

```python
ToolRetryMiddleware(
    retry_on=["RateLimitError", "TimeoutError"],
    max_retries=3,
    backoff_factor=0.5,        # 指数退避因子
    tools=["get_weather"],     # None = 全部工具
    on_failure="return_message",  # 'return_message' | 'raise' | callable
)
```

### 7. ModelRetryMiddleware（模型调用自动重试）
模型调用失败时自动重试，独立于 ToolRetry。

```python
ModelRetryMiddleware(max_retries=3, retry_on=["RateLimitError", "InternalServerError"])
```

与 ModelRetry 的差异：多了 `tools` 参数指定只对哪些工具生效。

### 7b. ModelFallbackMiddleware（模型回退）
主模型失败时自动切换到备选模型，按顺序尝试。

```python
ModelFallbackMiddleware(
    "gpt-5.4-mini",                   # 第一备选
    "claude-3-5-sonnet-20241022",    # 第二备选
)
```

---

## 三、限制与控制类（3个）

### 8. ModelCallLimitMiddleware（模型调用次数限制）
防止死循环或成本失控。

```python
ModelCallLimitMiddleware(
    thread_limit=10,        # 跨多次会话累计最多 10 次（需 checkpointer）
    run_limit=5,            # 单次 invoke 最多 5 次
    exit_behavior="end",    # 'end' 优雅结束；'error' 抛异常
)
```

### 9. ToolCallLimitMiddleware（工具调用次数限制）
比 Model Call Limit 更精细，可针对单个工具。

```python
ToolCallLimitMiddleware(thread_limit=20, run_limit=10),            # 全局
ToolCallLimitMiddleware(tool_name="search", thread_limit=5, run_limit=3),  # 单工具
```

`exit_behavior` 三种：`continue`（超限返回错误消息）、`error`（抛异常）、`end`（仅单工具场景）

### 10. HumanInTheLoopMiddleware（人机协同）
工具执行前暂停 Agent，等待人工批准/编辑/拒绝。

```python
HumanInTheLoopMiddleware(
    interrupt_on={
        "your_send_email_tool": "default",   # 需要审批
        "your_read_email_tool": False,       # 不需要
    },
    default_behavior=True,  # 未列出的工具默认也需要审批
)
```

⚠️ **必须配置 checkpointer**，暂停后靠检查点恢复状态。

---

## 四、工具与智能优化类（4个）

### 11. PromptChunkingMiddleware（提示词分块）
把大任务自动拆成子任务分步执行，Agent 在每个 chunk 间保持推理链条，适合复杂多步推理。

```python
PromptChunkingMiddleware(
    query="深入分析三个框架...",
    tools=[search_tool, file_tool],
)
```

### 12. LLMToolSelectorMiddleware（LLM 工具选择器）
工具太多时，用 LLM 从候选集中选出最相关的子集。

```python
LLMToolSelectorMiddleware(
    model="gpt-5.4-mini",
    max_tools=3,                    # 最多选 3 个
    always_include=["search"],      # 必选，不占配额
)
```

### 13. LLMToolEmulator（工具模拟器）
用 LLM 伪造工具返回结果，专为测试与原型设计。

```python
LLMToolEmulator()                          # 模拟全部工具
LLMToolEmulator(tools=["get_weather"])     # 只模拟指定工具
LLMToolEmulator(model="claude-sonnet-4-6") # 用指定模型生成模拟响应
```

### 14. ProviderToolSearchMiddleware（提供者侧工具搜索）
把部分工具推迟到提供者服务端搜索后暴露，模型按需发现工具。

⚠️ 需要支持服务端工具搜索的模型：Anthropic（Claude Sonnet 4+/Opus 4+/Haiku 4.5+）或 OpenAI（gpt-5.5+）

```python
ProviderToolSearchMiddleware(searchable_tools=["lookup_order"])
# 或在工具定义时标记 extras={"defer_loading": True}
```

---

## 五、文件系统与系统操作类（2个）

### 15. FilesystemFileSearchMiddleware（文件搜索）
注入 Glob 和 Grep 两个工具。

```python
FilesystemFileSearchMiddleware(
    root_path="/workspace",
    use_ripgrep=True,            # 优先 ripgrep
    max_file_size_mb=10,
)
```

### 16. ShellToolMiddleware（持久化 Shell 会话）
持久化 shell 会话，顺序执行多条命令。

⚠️ 安全：务必使用匹配的执行策略。不支持与 interrupt（人机协同）一起使用。

```python
ShellToolMiddleware(
    workspace_root="/workspace",
    execution_policy=HostExecutionPolicy(),   # 宿主机执行
    # 或 DockerExecutionPolicy(image="python:3.11-slim")
    startup_commands=["pip install requests"],
    redaction_rules=[RedactionRule(pii_type="api_key", detector=r"sk-[a-zA-Z0-9]{32}")],
)
```

**三种执行策略：** HostExecutionPolicy / DockerExecutionPolicy / CodexSandboxExecutionPolicy

---

## 六、子代理类（1个）

### 17. SubAgentMiddleware（子代理委派）
主 Agent 通过 `task` 工具委派子任务，上下文隔离——主 Agent 只拿到子代理的简洁结论。

```python
SubAgentMiddleware(
    default_model="claude-sonnet-4-6",
    default_tools=[],
    subagents=[{
        "name": "weather",
        "description": "This subagent can get weather in cities.",
        "system_prompt": "Use the get_weather tool...",
        "tools": [get_weather],
        "model": "gpt-5.5",
        "middleware": [],
    }]
)
```

支持用预编译的 LangGraph 图作为子代理（`CompiledSubAgent`）。主 Agent 始终有一个内置的 `general-purpose` 子代理。

---

## 七、选型速查表

| 问题域 | 推荐中间件 |
|--------|-----------|
| 对话过长撑爆上下文 | SummarizationMiddleware / ContextEditingMiddleware |
| 工具返回太长占用上下文 | FilesystemMiddleware + ContextEditingMiddleware |
| 跨会话长期记忆 | FilesystemMiddleware + CompositeBackend |
| 模型偶发失败 | ModelRetryMiddleware |
| 模型服务整体不可用 | ModelFallbackMiddleware |
| 外部 API 偶发失败 | ToolRetryMiddleware |
| 防止死循环/成本失控 | ModelCallLimitMiddleware + ToolCallLimitMiddleware |
| 高风险操作需人工审批 | HumanInTheLoopMiddleware（需 checkpointer） |
| 合规、脱敏、日志净化 | PIIMiddleware |
| 多步任务需规划与进度可见 | TodoListMiddleware |
| 工具太多影响准确度与 token | LLMToolSelectorMiddleware / ProviderToolSearchMiddleware |
| 调试时不想真调工具 | LLMToolEmulator |
| Agent 操控文件系统 | FilesystemMiddleware / FilesystemFileSearchMiddleware |
| Agent 执行命令 | ShellToolMiddleware |
| 任务委派、上下文隔离 | SubAgentMiddleware |

**实践建议：**
- 中间件可组合，一个生产 Agent 通常挂多个（Summarization + ToolRetry + ModelFallback + ToolCallLimit）
- 需状态的中间件要配置 checkpointer
- 涉及 Shell、PII、Human-in-the-loop 的场景沙箱运行
- 先用 LLMToolEmulator 验证行为再上线
