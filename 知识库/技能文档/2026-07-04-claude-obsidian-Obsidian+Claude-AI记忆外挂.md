# 又一个神级开源工具！用 Obsidian + Claude 整了个 AI 记忆外挂

> **来源**：极客之家（小黑）  
> **日期**：2026-07-04  
> **原文链接**：https://mp.weixin.qq.com/s/75pT77D5rn4J_HeYeQGkfw  
> **分类**：知识管理与归档

---

## 文章概要

开源项目 **claude-obsidian**（GitHub 8.5k⭐）：将 Obsidian 变为"自组织的 AI 第二大脑"。Claude Code 自动读取/归档/链接资料，生成 Markdown 知识图谱，配合 `hot.md` 热缓存实现跨会话记忆，支持自动研究循环。

**仓库**：https://github.com/AgriciDaniel/claude-obsidian

---

## 核心功能

### 自动归档与知识图谱
- 丢一个 PDF 进去，Claude 自动生成 8~15 个 wiki 页面，更新 index 和 log
- 所有内容纯 Markdown，平台无关
- Obsidian Graph view 可视化页面间链接关系（如 Transformer→Attention→KV Cache 自动连线）

### 热缓存与会话记忆（hot.md）
- 每次对话后，将近期上下文、关键结论、待办事项写入 `hot.md`
- 下次敲 `/wiki`，Claude 先读 `hot.md`，接着上次话题继续
- 解决 AI 插件"聊完就忘"的痛点

### 自动研究循环（/autoresearch）
- 给定主题，Claude 自主分解角度、搜索资料、抓取内容、综合写作、归档进 vault
- 默认三轮：第一轮大范围搜 → 第二轮找漏掉的 → 第三轮互相印证
- 自动创建实体页面（如"Pinecone"、"Milvus"、"Weaviate"）

### 画布可视化
- 将图片、PDF、wiki 页面、文字卡片丢到画布上，Claude 自动排版
- 可从最近生成的图片里抓素材直接上画布

### 多方法论支持（v1.8+）
- 四种知识组织方式：**LYT / PARA / Zettelkasten / Johnny Decimal**
- 通过 `bash bin/setup-mode.sh` 选择后，Claude 按该规则自动归档

### 其他特性
- 矛盾标记：自动标注矛盾并附来源
- Vault 维护：8 类 lint 检查
- 批量摄入：并行 agent 处理
- 多写作者安全：文件锁（v1.7+）
- 多模型支持：Claude / Gemini / Codex / Cursor / Windsurf

---

## 安装方式（三种）

### 方式一：克隆完整 vault（推荐）
```bash
git clone https://github.com/AgriciDaniel/claude-obsidian
cd claude-obsidian
bash bin/setup-vault.sh
```
打开 Obsidian → 选该文件夹为 vault → 在 Claude Code 敲 `/wiki`

### 方式二：Claude Code 插件
```bash
claude plugin marketplace add AgriciDaniel/claude-obsidian
claude plugin install claude-obsidian@agricidaniel-claude-obsidian
```

### 方式三：塞进现有 vault
复制 `WIKI.md` 到 vault 根目录，Claude Code 中贴提示词自动搭建。

---

## 对比同类

| 能力 | claude-obsidian | Smart Connections | Copilot | SystemSculpt |
|:---|:---:|:---:|:---:|:---:|
| 自动整理笔记 | ✅ | ❌ | ❌ | ❌ |
| 矛盾标记 | ✅ | ❌ | ❌ | ❌ |
| 会话记忆 | hot cache | ❌ | ❌ | ❌ |
| vault 维护 | 8类 lint | ❌ | ❌ | ❌ |
| 自主研究 | 3轮+归档 | ❌ | ❌ | ❌ |
| 方法论模式 | 4种 | ❌ | ❌ | ❌ |
| 画布可视化 | ✅ | ❌ | ❌ | ❌ |
| 多写作者安全 | 文件锁 | ❌ | ❌ | ❌ |
| 开源协议 | MIT | MIT | 免费增值 | 免费增值 |

---

## 建议读者

- ✅ 想搭建自增长知识库 → claude-obsidian
- ❌ 只想装个聊天机器人偶尔问两句 → Copilot 或 Smart Chat 更合适

---

## 关键词

`claude-obsidian` `Obsidian` `Claude Code` `AI第二大脑` `知识图谱` `hot.md` `自动研究` `autoresearch` `PARA` `LYT` `Zettelkasten` `知识管理` `自组织知识库`
