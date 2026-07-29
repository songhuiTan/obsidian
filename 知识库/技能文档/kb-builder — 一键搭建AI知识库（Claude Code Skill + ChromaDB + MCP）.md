# kb-builder — 一键搭建AI知识库（Claude Code Skill + ChromaDB + MCP）

> 来源：码哥（MageByte）｜公众号「码哥跳动」2026-06-20
> 原文：[0元使用 Claude Code 搭建个人的AI知识库](https://mp.weixin.qq.com/s/h063AWJq6AsTknqPpmLlDg)
> GitHub：[kb-builder](https://github.com/MageByte-Zero/kb-builder) | [awesome-ai-kb](https://github.com/MageByte-Zero/awesome-ai-kb)

## 概述

一套**完全本地运行、零成本的开源 AI 知识库方案**，面向不愿被商业 SaaS 月费绑架、对数据隐私敏感的技术人。

核心主张：**你不应该为"管理自己写过的笔记"按月付费**。

## 架构设计

**双 Repo 分离架构：**

```
awesome-ai-kb（内容仓库）
  ├── articles/
  │   ├── backend/        # Redis / MySQL / JVM / Kafka / 算法 / 架构等
  │   ├── wechat-ai/      # 公众号 AI 方向（Claude Code / MCP / Agent）
  │   └── workplace-ai/   # 职场 AI 提效（待扩充）
  ├── briefs/
  │   ├── papers/         # 论文速读
  │   ├── tools/          # 工具荐评
  │   └── trends/         # 趋势观察
  ├── community/          # AMA、Q&A
  ├── methodology/        # 写作规范、研究框架
  └── glossary.md         # AI + 后端技术术语表

kb-builder（工具仓库）：Claude Code Skill
  ├── SKILL.md            # Skill 定义（4种工作模式）
  ├── scripts/
  │   ├── config.yaml     # 内容源配置
  │   ├── install.sh      # 一键安装
  │   ├── index.py        # 索引管道（ChromaDB）
  │   └── search.py       # CLI 检索验证
  ├── mcp/
  │   └── kb_server.py    # FastMCP Server（3个工具）
  └── templates/
      └── kb-content-starter/   # 内容仓库模板
```

内容仓库只管「内容」，工具仓库只管「索引+检索」。内容永远是干净的纯文本 Markdown，工具出问题不影响内容。

## 技术栈

| 组件 | 选型 | 说明 |
|------|------|------|
| 向量数据库 | **ChromaDB** | 开源、零依赖、本地运行 |
| Embedding 模型 | **paraphrase-multilingual-MiniLM-L12-v2** | 384维，支持50+语言，中英文混查，MacBook上毫秒级 |
| 协议层 | **MCP (Model Context Protocol)** | 通过 FastMCP 暴露为 Claude Code 工具 |
| 编译工具 | **Claude Code Skill** | kb-builder 自身就是一个 SKILL.md |
| 分块策略 | **3种策略组合** | 结构感知800字 / 模板段落600字 / Q&A对 |

## 性能数据

- **内容库规模**：240 篇技术文章，覆盖 25+ 主题
- **索引规模**：240 篇文章 → 5877 个语义块（平均每篇 24 块）
- **检索命中率**：75%+（Top-3 中至少 2 条高度相关，50次测试）
- **查询延迟**：本地 800ms 以内
- **内存占用**：< 500MB（M1 MacBook Air）
- **花费**：0 元（全部开源/免费）

## MCP 工具（3个）

注册到 Claude Code 后可用的工具：

1. **`search_kb(query)`** — 语义搜索，输入自然语言，返回 Top-5 最相关的 chunk
2. **`list_kb_topics()`** — 查看知识库覆盖的主题分布
3. **`get_kb_stats()`** — 索引数据统计

## Skill 的 4 种工作模式

kb-builder 的 SKILL.md 定义了四种对话模式：

1. **全新搭建**（7步流程）：确认内容路径 → install.sh → 分块策略 → 首次索引 → 检索验证（5问≥4命中）→ MCP 接入 → 交付
2. **增量更新**：新内容 → 只跑 index.py 增量模式
3. **质量排查**：搜不到 → 诊断分块策略 / top_k / embedding
4. **打造我的知识库**：用自己的内容 → 配置 config.yaml

## 安装方式

```bash
git clone https://github.com/MageByte-Zero/kb-builder.git
cd kb-builder/scripts
bash install.sh
```

前置依赖：Python 3.10+、Git、Claude Code CLI。

## 战略分析

### 与现有工具链的对照

这套方案和我目前的 Hermes + Obsidian 知识库体系高度类似，技术选型上几乎是**同一思路的不同实现**：

| 维度 | kb-builder | 我当前的方案 |
|------|-----------|------------|
| 内容管理 | Obsidian 知识库 | Obsidian 知识库 |
| 检索 | ChromaDB + 本地 embedding | TencentDB Agent Memory |
| Skill 注册 | Claude Code SKILL.md | Hermes skills 系统 |
| MCP 接入 | kb_server.py (FastMCP) | Hermes native MCP |
| 增量索引 | index.py 增量模式 | 未实现（当前依赖 cron） |

### 差距与借鉴点

kb-builder 有几个值得关注的设计：

1. **双 Repo 分离** — 内容与工具严格分离，内容永远是纯 Markdown，工具可随时更换。我的 Obsidian 知识库已经做到了这一点，但 kb-builder 的 `templates/kb-content-starter/` 提供了一个可复用的内容仓库模板，值得参考。

2. **三种分块策略组合** — 结构感知（按 H2/H3 切）、模板段落（固定长度）、Q&A 对。这个分层设计比单一的固定 chunk 策略更合理。目前的 TencentDB Memory 没有做显式分块策略配置。

3. **SKILL.md 的 4 种工作模式** — 把技能定义成了清晰的状态机（搭建/增量/排查/自定义），每种模式有明确的步骤和验收标准。这是 SKILL.md 写作的一个优秀范例。

4. **检索验证环节** — "5 个测试问题，≥4 个命中才算通过"是一个可操作的验收标准。这比"感觉还行"要好得多。

### 整合可能性

kb-builder 可以作为一个**独立的 Hermes skill** 接入现有体系：

- 将 kb-builder 的脚本和 MCP server 抽取为 Hermes skill
- 内容源指向已有的 Obsidian 知识库
- 与 TencentDB Memory 形成互补：实时对话记忆 + 离线知识库检索

## 术语表

- **ChromaDB**：开源向量数据库，存储文本的 embedding 向量，支持语义检索
- **Embedding**：将文本转为向量（数字序列）的过程，语义相近的文本向量距离更近
- **MCP**：Model Context Protocol，AI 工具与外部世界之间的通用协议
- **分块策略 (Chunking Strategy)**：将长文本拆分为检索单元的方式，影响检索精度
- **双 Repo 分离**：内容仓库和工具仓库分离的设计模式

## 相关文档

- 技能文档/Claude Prime 一键告别重复 Prompt 工程.md（三层架构理念相通）
- 技能文档/Claude-Code-必备-skill-10分钟做出专业PPT、Excel分析和Word报告.md（同为 Claude Code Skill）
