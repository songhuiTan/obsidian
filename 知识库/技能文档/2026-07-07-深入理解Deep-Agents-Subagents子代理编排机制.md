# 深入理解 LangChain / Deep Agents 的 Subagents（子代理）编排机制

> **来源**：AnthroTech AI（微信公众号）
> **日期**：2026-07-07
> **原文链接**：https://mp.weixin.qq.com/s/1192zHlUT68ayKgMnOzN7Q

## 核心要点

Subagents 是 Deep Agents 解决"上下文膨胀"问题的关键机制。主 Agent 通过 `task` 工具委派工作，子代理在隔离上下文中完成专业任务，只返回最终结果。

**两大价值**：上下文隔离（context quarantine）+ 专门化指令/工具/模型。

## 默认子代理：general-purpose

每个 Deep Agent 自动拥有一个同步 `general-purpose` 子代理（可替换或禁用），它：
- 使用自己的默认 system prompt（可叠加 profile 覆盖）
- 继承主 Agent 的全部工具和模型
- 配置 skills 时继承主 Agent 的 skills

## 自定义 SubAgent（字典式）

| 字段 | 必需 | 说明 |
|------|------|------|
| `name` | ✅ | 唯一标识，主 Agent 通过此调用 `task()` |
| `description` | ✅ | 描述子代理做什么，要具体、面向动作 |
| `system_prompt` | ✅ | 子代理指令，**不继承** 主 Agent |
| `tools` | 可选 | 默认**继承**主 Agent；指定则**完全覆盖** |
| `model` | 可选 | 可覆盖主 Agent 模型 |
| `middleware` | 可选 | 追加到默认子代理栈 |
| `interrupt_on` | 可选 | 为特定工具配置 HITL |
| `skills` | 可选 | 自定义子代理**不继承**主 Agent skills |
| `response_format` | 可选 | 结构化输出 schema |
| `permissions` | 可选 | 文件系统权限，完全替换父代理 |

## 动态子代理（Beta）

配 interpreter 后可从**代码中**分派子代理——用循环、分支、并行批次把工作扇出到多个项。

```python
pip install -U "deepagents[quickjs]"
```

触发方式：请求中带"workflow"关键词（如"Run a workflow to review every file..."），内置 interpreter 系统提示把"workflow"视为通过解释器组织工作的信号。

## 结构化输出

`response_format` 让父代理收到合法 JSON 而非自由文本（需 `deepagents>=0.5.3`）。

```python
class ResearchFindings(BaseModel):
    summary: str
    confidence: float
    sources: list[str]

research_subagent = {
    "response_format": ResearchFindings,
    # ...
}
```

## 上下文传播

Runtime context 自动传播到所有子代理和工具。按代理定制可用命名空间键或独立字段。

```python
result = await agent.invoke(
    {...},
    context=Context(user_id="user-123", researcher_max_depth=3),
)
```

可用 `lc_agent_name` 元数据判断哪个子代理调用了工具。

## 最佳实践

| 原则 | 说明 |
|------|------|
| 描述要具体 | ✅ `"Analyzes financial data with confidence scores"` ❌ `"Does finance stuff"` |
| 提示要详尽 | 分步骤、输出格式清晰、字数受限 |
| 工具要精简 | 最小化工具集，按职责聚焦 |
| 模型要匹配 | 长文档用大上下文模型，数值分析用强推理模型 |
| 结果要简洁 | 只返回关键洞察，禁止原始数据和中间计算 |

## 排错速查

| 现象 | 解决 |
|------|------|
| 子代理不被调用 | description 更具体 + 主提示中明确指示用 task() 委派 |
| 上下文仍在膨胀 | 限字数+子代理写原始数据到文件系统，只返回分析摘要 |
| 选错子代理 | description 清晰区分，如 quick-researcher vs deep-researcher |
