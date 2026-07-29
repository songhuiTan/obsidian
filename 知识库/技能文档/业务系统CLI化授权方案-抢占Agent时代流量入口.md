---
source: 微信公众号
title: 业务系统 CLI 化授权方案：抢占 Agent 时代的流量入口
author: Anhui / 未来程式
date: 2026-07-05
category: AI Agent 框架与工程化
tags: CLI, 授权, OAuth, DeviceFlow, PAT, Agent, 架构设计
link: https://mp.weixin.qq.com/s/HTTdNVwLqhGIPnhfbKxKXw
status: 已归档
---

# 业务系统 CLI 化授权方案：抢占 Agent 时代的流量入口

> 作者：Anhui / 未来程式 | 2026-07-05
> 原文：[微信链接](https://mp.weixin.qq.com/s/HTTdNVwLqhGIPnhfbKxKXw)

---

## 核心观点

Agent 时代，CLI 是业务系统的「API 门面」和流量入口。谁先把自己的业务系统 CLI 化，谁就在未来的 Agent 生态里占了先机。而 CLI 化的第一个核心问题就是**授权方案**。

---

## 一、为什么 CLI 是最优连接方式

Agent 与业务系统交互的三种方式对比：

| 方式 | 优点 | 缺点 |
|------|------|------|
| RPA/UI 自动化 | 无需改造系统 | 脆弱（页面改就挂）、速度慢、不可控 |
| 直接调用后端 API | 原生能力 | Agent 需知道 API 文档、鉴权方式，门槛高 |
| **CLI 工具** | 输入简单、输出结构化、人+Agent 兼用 | 需要改造授权体系 |

**趋势信号：** 飞书、钉钉、瑞幸等大厂都在推 CLI 工具。

---

## 二、核心设计理念：隔离

网页端和 CLI 端使用场景完全不同：

| 维度 | 网页端 | CLI 端 |
|------|--------|--------|
| 使用者 | 人 | 脚本/Agent |
| 会话时长 | 短 | 长期有效 |
| 安全要求 | 单点互踢 | 不能影响网页端 |

→ 给 CLI **单独一套 Token 体系（PAT，Personal Access Token）**，与网页端 Session 完全分开。

---

## 三、后端改造方案

### 3.1 数据库设计（user_api_tokens）

```sql
CREATE TABLE IF NOT EXISTS `user_api_tokens` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `user_id` VARCHAR(64) NOT NULL COMMENT '用户ID',
  `token_name` VARCHAR(100) DEFAULT NULL COMMENT 'Token名称/描述',
  `token_hash` VARCHAR(64) NOT NULL COMMENT 'SHA-256 Token哈希值',
  `expires_at` DATETIME NOT NULL COMMENT '过期时间',
  `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_token_hash` (`token_hash`),
  KEY `idx_user_id` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

关键点：存 `token_hash`（SHA-256），有 `expires_at`，有 `token_name` 支持多 Token 管理。

### 3.2 Redis 临时状态（Device Flow）

| Key | 类型 | 说明 | TTL |
|-----|------|------|-----|
| `device_flow:device_code:<code>` | Hash | 设备状态（pending/approved）及临时 Token | 300s |
| `device_flow:user_code:<code>` | String | user_code → device_code 反向映射 | 300s |

### 3.3 三个核心接口

| 接口 | 路径 | 权限 | 说明 |
|------|------|------|------|
| 1. 申请设备码 | `POST /api/auth/device/code` | 匿名 | 返回 device_code / user_code / verification_uri |
| 2. 确认授权 | `POST /api/auth/device/approve` | 网页端登录 | 生成 CLI Token（`hwcli_tok_` 前缀），持久化 DB，更新 Redis |
| 3. 轮询获取 Token | `POST /api/auth/device/token` | 匿名 | 返回 `authorization_pending` 或 `accessToken` |

Token 格式：`hwcli_tok_` + `crypto.randomBytes(24).toString('hex')`，有效期 30 天。

### 3.4 网关鉴权拦截器（双轨制）

```
请求 → 提取 Bearer Token
  ├─ 以 "hwcli_tok_" 开头 → SHA-256 查 user_api_tokens 表验证
  └─ 否则 → 走原有网页端 Session 鉴权逻辑
```

网页端和 CLI 端完全解耦，互不干扰。

---

## 四、前端改造

- 新增 `/device` 授权页面，从 URL 读取 `code` 自动填充
- 未登录时保存当前 URL 到 `sessionStorage`，登录后自动跳回
- 授权成功后提示用户返回终端

---

## 五、CLI 端设计

### 三层架构
1. **命令层** — 用户交互
2. **服务层** — 逻辑编排
3. **API 层** — HTTP 调用

### 本地配置存储
`~/.config/configstore/huiwsper.json`

```typescript
interface SystemConfig {
  baseUrl: string;
  auth?: SystemAuth;
  cachedToken?: string;
  tokenExpiresAt?: string;
  refreshToken?: string;
}
```

### 三种认证模式

| 模式 | 适用场景 | 说明 |
|------|---------|------|
| **API Key/PAT** | CI/CD、机器调用 | 后台生成 Token，无自动刷新 |
| **Password** | 传统系统 | 密码本地 base64 混淆存储，Token 过期自动重登 |
| **Device Flow** | 企业 SSO（推荐） | 浏览器授权，密码不落地，支持 Refresh Token 静默刷新 |

### 401 失效自愈机制

- **主动检查**：每次请求前检查 Token 有效期，5 分钟内即将过期且有 Refresh Token 则主动刷新
- **被动重试**：响应拦截器捕获 401 → 自动刷新 Token → 重试原请求，第二次失败才报错

整个过程用户完全无感。

---

## 六、方案特点

1. **安全性**：Token 哈希存储，支持过期，密码不落地
2. **隔离性**：CLI 和网页端完全独立，互不踢下线
3. **易用性**：支持 Device Flow 浏览器授权，体验流畅
4. **扩展性**：未来可加 Token 权限控制、管理页面、审计日志等

---

## 关键启发

- CLI 是 Agent 时代的**结构化接入层**，对比 RPA 和裸 API 有天然优势
- **双轨 Token 体系**（Session + PAT）解决人机混用的会话冲突核心痛点
- Device Flow（OAuth 2.0 设备授权流程）是 CLI 授权的最佳实践参考
- 401 自愈机制是 CLI 用户体验的关键细节
