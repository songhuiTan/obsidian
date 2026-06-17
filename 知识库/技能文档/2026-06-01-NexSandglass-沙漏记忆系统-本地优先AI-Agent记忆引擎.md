---
title: "NexSandglass（沙漏记忆系统）- 本地优先 AI Agent 记忆引擎"
source: "lovevin1314-tech"
source_url: "https://github.com/lovevin1314-tech/NexSandglass-Agent-DedicatedMemory"
date: "2026-06-01"
tags: [NexSandglass, Agent 记忆, 本地优先, MCP, 记忆引擎, 开源]
---

**备份路径**：`知识库/assets/backup-github/NexSandglass-Agent-DedicatedMemory/`（完整 `--mirror` 克隆）

**项目简介**：NexSandglass 是一个本地优先的 AI Agent 记忆引擎，纯 Python stdlib + SQLite，零外部依赖。支持四路并发搜索（FTS5 · IDX · TF-IDF · Shadow Sand），具备偏移率追踪、灵魂蒸馏、情绪感知等高级记忆能力。

---

## 核心能力

| 维度 | 说明 |
|------|------|
| **偏移率（Drift Velocity）** | 追踪决策随时间的变化方向和幅度 |
| **织线知识图谱** | 从对话中提取实体关系三元组，无需 LLM |
| **织布机（Weave Machine）** | 将决策连接为因果链 |
| **灵魂蒸馏（Soul Distillation）** | 从累积偏移中构建动态画像 |
| **情绪感知** | 会话级情绪熵摘要 |
| **零依赖** | 纯 Python stdlib + SQLite |

## 与现有方案对比

| 维度 | Mem0 / Letta | NexSandglass V2.9 |
|------|------|------|
| 依赖 | 向量数据库 + 多个包 | ✅ **零依赖，纯 stdlib** |
| 注入量 | ~200-22000 token | ✅ **~60 token（236字符）** |
| 模块化 | 单体 | ✅ **33 模块** |
| 决策追踪 | ❌ | ✅ 决策粒子 + 偏移率 + 心理预判 |
| 阶段感知 | ❌ | ✅ 自动切阶段 |
| 情绪感知 | ❌ | ✅ 情绪熵 |
| 画像溯源 | ❌ | ✅ SHA256 密码学验证 |
| 搜索 | 向量检索 | ✅ **四路并发**（影子沙+FTS5+IDX+TF-IDF） |
| 知识图谱 | ❌ | ✅ 织线三元组（零 LLM） |
| 安装 | 服务栈 | ✅ 一行 `python install.py` |

## 记忆层架构（L0-L3）

| 层 | 操作 | median | p99 |
|----|------|--------|-----|
| **L1 写** | 单次落沙 | 4.3ms | 19.5ms |
| **L2 搜** | FTS5搜索 | 1.6ms | 5.4ms |
| | 四路并发 | 79.4ms | — |
| | 影子沙 | 0.7ms | 1.2ms |
| **L3 思** | 综合偏移率 | <0.1ms | — |
| | 情绪熵 | 6.5ms | — |
| | 织布洞察 | 85.3ms | — |

## 设计原则

1. **层追加不替换** — 新层叠加，永不修改已定稿的下层
2. **本地优先，LLM 增强** — 没 API Key 一样能跑
3. **决策是链条不是单点** — A→B→C→回到A
4. **改了A必须同步B** — 改名/改签名后全项目 grep
5. **极简注入** — 每轮~60token，LLM按需 `sandglass_search`
6. **零外部依赖** — Python stdlib + SQLite

## Hermes 集成

```bash
curl -sSL https://raw.githubusercontent.com/lovevin1314-tech/NexSandglass-Agent-DedicatedMemory/main/remote_install.py | python
```

然后在 Hermes 的 `.env` 写入 `NEXSANDBASE_HOME=~/.neurobase`，重启 Gateway 即刻生效。

## 项目文件结构

仓库包含 50+ 文件，核心模块：

- `sandglass_log.py` / `sandglass_vault.py` / `sandglass_think.py` — 核心记忆引擎
- `sandglass_mcp.py` — MCP 服务端
- `memory_provider.py` — 记忆提供者
- `l0_buffer.py` / `l3_*.py` — L0-L3 各层实现
- `emotion_l3.py` / `emotion_vocab.py` — 情绪系统
- `offset_l3.py` / `offset_signals.py` — 偏移率系统
- `weave_l3.py` / `weavethread.py` — 织线/织布机
- `persona_l3.py` / `l3_persona.py` — 画像系统
- `decision_particles.py` — 决策粒子
- `pulse.py` / `heartbeat.py` — 心跳与脉冲
- `shadow_sand.py` — 影子沙搜索
- `soul_diff.py` — 灵魂差异
- `nightwatch.py` — 夜间守望
- `偏移率说明书.md` — 中文文档
