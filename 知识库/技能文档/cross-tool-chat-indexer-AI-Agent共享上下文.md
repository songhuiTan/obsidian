# cross-tool-chat-indexer：让所有 AI Agent 共享上下文

> 来源：九歌AI · 2026-07-10
> 原文：https://mp.weixin.qq.com/s/ELWydWM8gBq5kI0o9aYW4A
> 分类：工具与实用技能

---

> 开源 Skill，统一索引 Codex / Claude Code / WorkBuddy 三个 AI 编码工具的聊天历史，SQLite FTS5 全文检索 + 项目级上下文召回，终结"开新对话就失忆"循环。

**GitHub**：https://github.com/JiugeLi/JiugeSkills/tree/main/cross-tool-chat-indexer

---

## 解决的核心问题

AI 编码助手越来越强，但共同的硬伤：**没有连续的记忆**。

- **工具间断裂**：Codex 里聊的，Claude Code 不知道；WorkBuddy 里做的，Codex 看不到
- **会话间断裂**：开新对话就是一张白纸，上周的架构决策/踩坑记录全部清零
- **脑断裂**：自己也记不清"上次在哪聊的、聊了什么"

---

## 架构：四阶段流水线

### 1️⃣ 抽取
三个工具各写一个解析器：
- **Codex**：读 `~/.codex/sessions/` jsonl 事件流
- **Claude Code**：读 `~/.claude/projects/` 项目目录下的 jsonl
- **WorkBuddy**：读本地 SQLite 元信息 + AI 生成文件
- 每条记录额外记录 `cwd`（工作目录），这是项目级召回的关键

### 2️⃣ 归一化
三个来源格式统一成同一张表。不管记忆从哪来，进了这张表都是同一种格式。

### 3️⃣ 存储
写入 SQLite FTS5，建两张虚拟表：
- `fts_unicode`：unicode61 分词 + 中文按字分词（汉字间插空格强制单字索引）
- `fts_trigram`：trigram 分词器，三字符滑窗切分，子串匹配更精准
- 两张表并集检索

### 4️⃣ 检索与召回
- `search`：特殊字符处理 → 两张 FTS 表同时搜 → 结果并集
- `recall`：额外做路径匹配 + 内容匹配 + 合并排序 + 压缩输出

---

## 三条命令

```bash
# 统一索引——三个工具历史全部抽取、归一化、写入 FTS5
python scripts/indexer.py index --source all

# 跨工具检索——不用管在哪个工具聊的，直接搜
python scripts/indexer.py search "wireguard 配置"

# 项目级上下文召回——开新对话前把记忆续上
python scripts/indexer.py recall "/path/to/project" --top 5
```

`recall` 输出压缩成「意图 + 关键问答」精简卡片，支持 `--format md` 直接输出 Markdown，贴进新对话的 system prompt——AI 就能"想起来"之前在这个项目上做过什么。

---

## 中文搜索方案

SQLite FTS5 默认分词器 `unicode61` 把连续中文当成一个整体 token。解决方案：**在每个汉字之间插空格，强制按字分词**。

"淘宝排名优化方案" → "淘 宝 排 名 优 化 方 案"

实测效果：

| 搜索词 | 命中 |
|--------|------|
| "淘" | ✅ 单字生效 |
| "淘宝" | ✅ 命中 |
| "wireguard" | ✅ 英文正常 |
| "wireguard 配置" | ✅ 中英混合 OR 语义 |
| "K3S" | ✅ 跨源命中 Codex + Claude Code |

---

## 真实数据

来自作者本机 466 个独立会话：

| 指标 | 数值 |
|------|------|
| 总索引记录数 | 25,323 条 |
| 独立会话数 | 466 个 |
| 索引库大小 | 190 MB |
| Codex | 22,405 条 |
| Claude Code | 2,733 条 |
| WorkBuddy | 185 条 |

增量索引验证：从零重建 vs 再跑一次，数据一致不重复不丢失。

---

## 边界说明

**能做到：** 统一记忆、中文检索、项目级召回、增量索引、零依赖（纯 Python 标准库）

**做不到：**
1. WorkBuddy 聊天正文在腾讯云端，本机只有标题和生成物文件
2. 多词查询是 OR 语义（需精排得自己后处理）
3. 关键词检索而非语义向量检索（需相似度得接 embedding 层）

---

## 安装

```bash
git clone https://github.com/JiugeLi/JiugeSkills.git

# WorkBuddy
cp -r JiugeSkills/cross-tool-chat-indexer ~/.workbuddy/skills/

# Codex
cp -r JiugeSkills/cross-tool-chat-indexer ~/.codex/skills/

# Claude Code
cp -r JiugeSkills/cross-tool-chat-indexer ~/.claude/skills/

# 构建索引
python cross-tool-chat-indexer/scripts/indexer.py index --source all
```

前提：Python 3.10+，零第三方依赖。索引只读本地文件，不修改源数据，不联网。
