# 飞书API访问方法

## 概述

本文档记录如何使用飞书开放API访问飞书文档（wiki、docx），无需依赖第三方技能包。

## 快速开始

### 1. 获取访问令牌

```bash
curl -s "https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal" \
  -H "Content-Type: application/json" \
  -d '{
    "app_id": "cli_a92c13333ab89cd3",
    "app_secret": "7S1zNeGbg0SVygcPdnSyshtE83Knsqzp"
  }'
```

返回示例：
```json
{
  "code": 0,
  "expire": 6592,
  "msg": "ok",
  "tenant_access_token": "t-g10439haSJJCTXTW7675KER6RDQXEVEHZRMGFVGT"
}
```

### 2. 从URL提取token

- **Wiki链接**：`https://xxx.feishu.cn/wiki/{token}`
- **Docx链接**：`https://xxx.feishu.cn/docx/{token}`
- 直接提取 `{token}` 部分

### 3. 读取文档内容

```bash
# 获取文档基本信息
curl -s "https://open.feishu.cn/open-apis/docx/v1/documents/{token}" \
  -H "Authorization: Bearer {tenant_access_token}"

# 获取文档完整内容
curl -s "https://open.feishu.cn/open-apis/docx/v1/documents/{token}/blocks" \
  -H "Authorization: Bearer {tenant_access_token}"
```

## 常用API端点

| 端点 | 用途 |
|------|------|
| `/open-apis/auth/v3/tenant_access_token/internal` | 获取访问令牌 |
| `/open-apis/wiki/v2/spaces` | 列出知识空间 |
| `/open-apis/wiki/v2/nodes/{token}` | 获取wiki节点详情 |
| `/open-apis/docx/v1/documents/{token}` | 获取文档信息 |
| `/open-apis/docx/v1/documents/{token}/blocks` | 获取文档内容块 |
| `/open-apis/drive/v1/files/{file_token}/download_url` | 获取文件下载链接 |

## 重要提示

### Token有效期
- `tenant_access_token` 有效期约2小时
- 过期后需要重新获取

### Wiki vs Docx的区别
- Wiki链接通常包含 `/wiki/` 路径
- 但实际内容可能使用docx API访问
- **关键技巧**：如果wiki API返回404，尝试使用docx API

### 权限要求
需要在飞书开放平台配置以下权限：
- `wiki:wiki` 或 `wiki:wiki:readonly`
- `docx:document:readonly`
- `docx:document.block:convert`

### 附件限制
- API无法直接下载附件
- 需要手动在网页端打开文档处理

## 实战案例

### 案例1：访问OpenClaw新手教程

**文档链接**：`https://lcn32lxqhc07.feishu.cn/wiki/QsSAwtXibiowgokAVXhcuupAnMe`

**Token**：`QsSAwtXibiowgokAVXhcuupAnMe`

**执行步骤**：
1. 获取tenant_access_token
2. 使用docx API访问文档blocks
3. 解析返回的JSON数据
4. 成功获取11个必装技能的详细说明

### 案例2：访问HTTP请求技能文档

**文档链接**：`https://s24u7xr5dc.feishu.cn/wiki/LHAUwB2cei58AwkWkUWckSx6nK3`

**Token**：`LHAUwB2cei58AwkWkUWckSx6nK3`

**执行结果**：
- 成功获取文档信息
- 发现包含files.zip附件（无法通过API下载）
- 文档本身只有标题，内容在附件中

## 飞书凭据

已配置的飞书应用凭据：
- **App ID**：`cli_a92c13333ab89cd3`
- **App Secret**：`7S1zNeGbg0SVygcPdnSyshtE83Knsqzp`
- **配置位置**：`/Users/songhuitan/.openclaw/openclaw.json`

## 相关技能

OpenClaw新手必装10 Skills中与飞书相关的：

| 技能 | 安装命令 | 作用 |
|------|---------|------|
| feishu-lark | `npx skills add openclaudia/openclaudia-skills@feishu-lark -g -y` | 飞书协同文档同步 |

**注意**：安装的feishu-doc和feishu-wiki技能包只包含SKILL.md文档，没有实际工具实现。需要直接使用本文档提供的API方法。

## 工作流总结

```
1. 用户请求访问飞书文档
       ↓
2. 提取URL中的token
       ↓
3. 获取tenant_access_token
       ↓
4. 尝试docx API访问文档blocks
       ↓
5. 解析JSON数据，提取内容
       ↓
6. 返回格式化的文档内容
```

## 常见问题

### Q: 为什么不使用feishu技能包？
A: 安装的技能包只有说明文档，没有实际工具实现。直接使用飞书官方API更稳定可靠。

### Q: wiki API返回404怎么办？
A: 尝试使用docx API。很多wiki链接的底层内容都是通过docx API访问的。

### Q: 附件怎么处理？
A: API无法直接下载附件。需要告诉用户手动在网页端打开文档，下载附件。

### Q: Token过期了怎么办？
A: 重新调用tenant_access_token接口获取新的token。

## 相关文档

- [飞书开放平台文档](https://open.feishu.cn/document/server-docs/docs/server-docs/overview/introduction)
- [[OpenClaw新手必装10 Skills]]
- [[2026-03-09会话记录]]

---

**创建时间**：2026-03-09
**最后更新**：2026-03-09
**标签**：#飞书 #API #OpenClaw #技术文档
