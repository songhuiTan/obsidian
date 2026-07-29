# CoreCoder：想看懂 Claude Code 底层，别上来就啃 50 万行

> 来源：公众号「Github开源项目」（作者：东哥）
> 时间：2026-07-01
> 归档：2026-07-02
> GitHub：he-yufeng/CoreCoder

## 项目定位

CoreCoder 是一个轻量级开源编码 Agent，旨在让开发者理解 Claude Code 的核心架构，而非替代。

**核心数据：** 1,714 行物理代码（18 个文件），其中 engine 核心仅 **1,081 行**。

## 核心能力

保留了 Claude Code 最关键的底层机制：

- **主循环**（agent.py）
- **模型接口**（llm.py）
- **上下文管理**（context.py）
- **工具系统**（tools/bash.py 等）
- **Session 管理**
- **并行工具执行**
- **三层 context 压缩**
- **bash 危险命令拦截**（regex blacklist）
- **文件读写、shell 执行、sub-agent 调度**

## 配置与兼容性

- 默认走 OpenAI-compatible API
- DeepSeek/Ollama 等换 `OPENAI_BASE_URL` + 模型名即可
- 不兼容的通过 LiteLLM 接入 100+ 服务商

## 学习建议

1. 从 `agent.py`（主循环）开始
2. 接着看 `llm.py`（模型怎么要工具）、`context.py`（上下文不爆）、`tools/bash.py`
3. 先不加 MCP、RAG 等外围模块
4. 核心看：模型怎么要工具 → 工具怎么回填 → 上下文怎么不爆

## 注意

危险命令拦截基于 regex blacklist，非安全沙箱。接不可信输入仍需容器隔离+权限控制。

## 相关链接

- GitHub：https://github.com/he-yufeng/CoreCoder
- 原文：https://mp.weixin.qq.com/s/NC2h55_-6SOKdRMQnXA43g
