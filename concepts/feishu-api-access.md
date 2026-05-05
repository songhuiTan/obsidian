---
title: 飞书文档访问方法
created: 2026-05-05
updated: 2026-05-05
type: concept
tags: [integration, tool, api, workflow]
sources:
  - 知识库/技能文档/飞书API访问方法.md
  - productivity/feishu-doc/SKILL.md
  - productivity/feishu-wiki/SKILL.md
confidence: high
---

# 飞书文档访问方法

## 概述

读取飞书文档有三种路径：**Hermes 内置 Skill**（推荐）、**直接 API 调用**、**OpenClaw 技能包**。本文覆盖全部三种。

所有路径都依赖同一个飞书开放平台凭据，已记录在知识库中。

---

## 路径一：Hermes feishu-doc Skill（推荐）

Hermes Agent 内置了 `feishu-doc` 工具，通过 `feishu_doc` 单一接口操作飞书文档。

### 读取文档

```json
{ "action": "read", "doc_token": "ABC123def" }
```

从 URL 提取 token 的规则：
- `https://xxx.feishu.cn/docx/ABC123def` → `ABC123def`
- `https://xxx.feishu.cn/docs/doccn123c` → `doccn123c`

返回：标题、纯文本内容、块统计信息。

### 读取结构化内容（表格、图片等）

```json
{ "action": "list_blocks", "doc_token": "ABC123def" }
```

当 `read` 返回的 `hint` 字段提示有表格/图片等结构化内容时，用 `list_blocks` 获取完整块数据。

### 写入/创建文档

```json
# 写入（替换全部内容）
{ "action": "write", "doc_token": "ABC123def", "content": "# Title\n\nMarkdown..." }

# 创建+写入（一步到位，推荐）
{ "action": "create_and_write", "title": "新文档", "content": "# 标题\n\n内容..." }

# 追加内容
{ "action": "append", "doc_token": "ABC123def", "content": "追加内容" }
```

### 评论操作

```json
# 列出评论
{ "action": "list_comments", "doc_token": "ABC123def" }

# 创建评论
{ "action": "create_comment", "doc_token": "ABC123def", "content": "评论内容" }
```

### 完整读取工作流

```
read (获取纯文本 + 统计)
  ↓
检查返回中的 block_types（Table, Image, Code...）
  ↓
如有结构化内容 → list_blocks 获取完整数据
```

### 依赖配置

需在 Hermes 配置中启用（默认已启用）：
```yaml
channels:
  feishu:
    tools:
      doc: true
```

权限要求：`docx:document`、`docx:document:readonly`、`docx:document.block:convert`、`drive:drive`

---

## 路径二：feishu-wiki Skill（知识空间导航）

配合 `feishu-doc` 使用，用于浏览飞书知识空间结构。

### 列出知识空间
```json
{ "action": "spaces" }
```

### 列出节点
```json
{ "action": "nodes", "space_id": "7xxx" }
```

### 获取节点详情 → 读取内容
```json
# 1. 获取节点详情（返回 obj_token）
{ "action": "get", "token": "wiki_token" }

# 2. 用返回的 obj_token 调用 feishu_doc 读取
feishu_doc { "action": "read", "doc_token": "obj_token" }
```

这个流程是：`feishu_wiki` 负责导航发现文档，`feishu_doc` 负责读写内容。

---

## 路径三：直接飞书 API 调用（curl）

适用于没有 Hermes 环境的场景，或者需要精细控制 API 参数的场景。

### 1. 获取访问令牌

```bash
curl -s "https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal" \
  -H "Content-Type: application/json" \
  -d '{
    "app_id": "cli_a92c13333ab89cd3",
    "app_secret": "7S1zNeGbg0SVygcPdnSyshtE83Knsqzp"
  }'
```

返回 `tenant_access_token`（有效期约2小时）。

### 2. 读取文档内容

```bash
# 文档基本信息
curl -s "https://open.feishu.cn/open-apis/docx/v1/documents/{token}" \
  -H "Authorization: Bearer {tenant_access_token}"

# 完整内容块
curl -s "https://open.feishu.cn/open-apis/docx/v1/documents/{token}/blocks" \
  -H "Authorization: Bearer {tenant_access_token}"
```

### 常用 API 端点

| 端点 | 用途 |
|------|------|
| `/open-apis/auth/v3/tenant_access_token/internal` | 获取访问令牌 |
| `/open-apis/wiki/v2/spaces` | 列出知识空间 |
| `/open-apis/wiki/v2/nodes/{token}` | 获取 wiki 节点详情 |
| `/open-apis/docx/v1/documents/{token}` | 获取文档信息 |
| `/open-apis/docx/v1/documents/{token}/blocks` | 获取文档内容块 |
| `/open-apis/drive/v1/files/{file_token}/download_url` | 获取文件下载链接 |

---

## 路径四：OpenClaw feishu-lark Skill

OpenClaw 生态中有对应的技能包，但**仅包含 SKILL.md 说明文档，没有实际工具实现**。实际读写仍需通过 API 或 Hermes Skill。

```bash
npx skills add openclaudia/openclaudia-skills@feishu-lark -g -y
```

---

## 路径对比

| 维度 | Hermes feishu-doc | 直接 API | OpenClaw 技能包 |
|------|------------------|---------|----------------|
| 上手难度 | 一条指令 | 需 curl + token 管理 | 安装后仍需 API |
| 结构化内容 | 支持（list_blocks） | 支持 | 无工具实现 |
| 写入 | 支持 | 需自行实现 | 无工具实现 |
| 评论管理 | 支持 | 需自行实现 | 无工具实现 |
| 知识空间导航 | feishu-wiki 配合 | 手动 | 无工具实现 |
| 推荐场景 | **日常使用** | 自动化脚本/调试 | 已弃用 |

## 实战技巧

- **Wiki 链接 404 怎么办**：很多 wiki 链接底层走的是 docx 接口，改用 feishu-doc 或 docx API 读取
- **附件无法下载**：API 不支持附件下载，需手动在网页端操作
- **Token 过期**：2小时有效期，需重新获取
- **凭据位置**：App ID/Secret 已记录在知识库 `飞书API访问方法.md`，同时也配置在 `~/.openclaw/openclaw.json`
