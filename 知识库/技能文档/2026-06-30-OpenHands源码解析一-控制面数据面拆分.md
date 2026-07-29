---
title: "OpenHands 源码解析（一）：那个最火的开源 AI 程序员，正在「拆掉」自己"
author: "zhanglongyanmany (小张学AI Agent)"
source_url: "https://mp.weixin.qq.com/s/2alDVCA3VVb5n3ghpfqwDw"
date: "2026-06-30"
tags: [OpenHands, 源码解析, Agent Canvas, 控制面, 数据面, Agent架构, ACP, MCP, 沙箱]
---

# OpenHands 源码解析（一）：那个最火的开源 AI 程序员，正在「拆掉」自己

> OpenHands（78K+ Star）已不再把自己描述成"一个会写代码的 AI"，而是写成了 **self-hosted developer control center**。它可以运行 OpenHands、Claude Code、Gemini 等第三方 agent，或任何 ACP-compatible agent。

更关键的是：**`The code in this repo is moving!`**

- OpenHands Agent 和 Agent Server 源码 → `OpenHands/software-agent-sdk`
- Agent Canvas 源码 → `OpenHands/agent-canvas`

这意味着：这个仓库正在从"单体 AI 程序员"变成一个 **Agent Canvas 控制中心**。真正执行代码、跑测试、调用工具的部分正在被搬到数据面；留下的核心是会话、沙箱、事件、密钥、MCP、设置、集成这些控制面能力。

## 一、README 已经把坐标系改了

README 现在讲的是 Agent Canvas：把 coding agents 变成 self-hosted、always-on 的 engineering team。它能默认运行 OpenHands agent，也可以使用 Claude Code、Gemini 等第三方 agent。

如果继续按旧思路读 OpenHands，很容易在仓库里找"agent 到底怎么思考、怎么改代码"。但现在更重要的问题是：
- 这个仓库如何创建一场对话？
- 如何选择 docker / process / remote 沙箱？
- 如何把用户的 LLM 配置、skills、MCP、secrets 装配成一次启动请求？
- 如何接收 agent-server 回流的事件？
- 如何让一个不可信的执行环境用上 GitHub token、Tavily API key，却拿不到底牌？

这就是**控制面 / 数据面分离**。

## 二、app_server 是控制面，不是 agent 内核

`openhands/app_server/` 下约 33,523 行 Python。目录结构：

```
app_conversation/  — 会话元数据、启动任务、状态查询、模型切换
sandbox/           — docker / process / remote 数据面后端
event/             — 事件落库
event_callback/    — 回调处理
mcp/               — 服务端工具代理
secrets/           — 用户配置与密钥边界
settings/
user/
git/
integrations/
web_client/
```

这些模块不像一个"agent 运行时"，更像一个**平台控制层**。

旧入口 `openhands/server/listen.py` 只剩兼容层：
```python
from openhands.app_server.app import app
__all__ = ['app']
```
文件开头写着 deprecated，建议直接使用 `openhands.app_server.app`。

## 三、执行内核已经以依赖形式接回来

`pyproject.toml` 里有三行：
```
openhands-agent-server==1.29.0
openhands-sdk==1.29.0
openhands-tools==1.29.0
```

默认 agent-server 镜像：
```
AGENT_SERVER_IMAGE = 'ghcr.io/openhands/agent-server:1.29.0-python'
```

创建会话时，启动链路会先等沙箱启动，拿到 `agent_server_url`，校验 agent-server SDK 版本，构造 `StartConversationRequest`，最后带 `X-Session-API-Key` POST 到 `{agent_server_url}/api/conversations`。**这不是函数调用，是跨边界启动。**

## 四、三条回路把拆开的系统接起来

### 第一条：Start 请求

`AppConversationStartTask` 列出 8 个状态：
```
WORKING → WAITING_FOR_SANDBOX → PREPARING_REPOSITORY → RUNNING_SETUP_SCRIPT
→ SETTING_UP_GIT_HOOKS → SETTING_UP_SKILLS → STARTING_CONVERSATION → READY → ERROR
```

启动会话可能很慢（可能涉及启动沙箱），所以要踢一个 background task。

### 第二条：事件回流

agent-server 执行过程中产生事件，通过 webhook 回到 app_server：
```python
@router.post('/events/{conversation_id}')
async def on_event(events: list[Event], conversation_id: UUID, ...)
```
做三件事：保存事件 → 从 stats/execution_status 事件更新会话元数据 → 后台跑 callback processor。

### 第三条：能力与密钥（最巧妙）

OpenHands 需要让沙箱里的 agent 用上 GitHub token、Tavily 搜索、MCP 工具，但又不能把所有底牌直接交给一个会执行代码的环境。

处理 git provider token 时，它不是直接把明文 token 塞进请求，而是生成一个 **JWS access token**，然后构造 `LookupSecret` 让沙箱去"兑换"：

```python
LookupSecret(
    url=web_url + '/api/v1/webhooks/secrets',
    headers={'X-Access-Token': access_token},
    description=description,
)
```
**沙箱拿到的是"去哪兑换"的票，不是 token 本身。**

MCP 也类似。Tavily MCP proxy 的注释直接说：让沙箱使用 Tavily search，但不暴露 API key。默认 MCP server 配成 `{web_url}/mcp/mcp`，再带上 conversation id 和 session key。这就是**反向 MCP**：工具能力看起来在沙箱里可用，但关键凭证和服务端行为留在 app_server。

## 五、它不是"变薄"，而是边界变硬

`app_server` 仍然有三万多行 Python，复杂度没有消失，只是换了位置：
- **生命周期复杂度**：一次对话要经历启动任务、沙箱状态、agent-server 状态、pending message、归档删除
- **安全复杂度**：密钥要能用但不能随便回显；MCP 要能调但 API key 不能进沙箱
- **数据复杂度**：事件要落存储，状态要回填，回调要后台跑
- **扩展复杂度**：同一套 app_server 要服务 OSS、本地、远端、云平台和企业版

## 六、亮点和隐患都在边界上

**亮点：** 边界清楚以后，OpenHands 可以同时接 OpenHands Agent、Claude Code、Gemini 等第三方 agent，甚至任何 ACP-compatible agent。不同 agent 共享前端、会话、沙箱、事件、密钥、MCP、集成这些平台能力。

**隐患：**
1. **版本漂移** — 控制面和数据面分开后，版本兼容从"代码仓库内部问题"变成"跨服务协议问题"。源码里专门有 agent-server 版本校验
2. **启动链路变长** — 从创建 task、等沙箱、准备 workspace、装配 request 到 POST agent-server，每一步都可能失败
3. **安全边界** — LookupSecret、反向 MCP、session key 都在降低明文泄漏面，但 fallback、调试开关、兼容逻辑处理不好，边界就会变软

## 结语

OpenHands 已经不是"一个 AI 程序员"的单点故事。它更像一个**控制中心**：把不同 agent、不同沙箱、不同事件源、不同凭证边界，组织成一套可运行的平台。

---

*本系列后续12篇将依次拆解：对话生命周期 → 沙箱后端 → event sink → 安全边界 → 前端实时系统 → 企业版*
