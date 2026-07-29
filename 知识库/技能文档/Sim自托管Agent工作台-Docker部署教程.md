---
source: 微信公众号
author: 几何构造师
account: 智研纪
date: 2026-06-25 06:54
url: https://mp.weixin.qq.com/s/tzXWy0vn0IR_P1GrxWEnFw
tags: Sim, SimStudio, Agent工作台, Docker, 自托管, Ollama, 知识库, 工作流
---

# 手把手教你自托管一个 Agent 工作台：用 Sim 跑起你的知识库和工作流

> 项目地址：https://github.com/simstudioai/sim
> 官方描述：open-source AI workspace where teams build, deploy, and manage AI agents
> 一句话定位：**先有一个控制台，再谈多个 Agent 协作。**

## 🔍 它解决的是工作台问题

Sim 把聊天、文件、知识库、表格、工作流放在同一个 workspace 里。核心能力：

- ✅ 用自然语言让系统帮你搭 Agent
- ✅ 基于工作区数据生成文档
- ✅ 上传资料，让 Agent 依据自己的内容回答
- ✅ 把结构化数据放进表格
- ✅ 在可视化画布上连接步骤

**不适合：** 只想本地聊天、机器资源紧张、不愿意碰 Docker。

## 🧰 部署前准备

### ① Docker
```bash
docker --version
docker compose version
docker info
```

### ② 端口检查
最小访问入口：`http://localhost:3000`
WebSocket：`3002`，PostgreSQL：`5432`
```bash
ss -lntp | grep -E '3000|3002|5432' || true
```

### ③ 内存和磁盘
- **12GB+ RAM**（官方要求，8GB 机器 Docker 启动慢、容器异常退出正常）
- **16GB 内存起步更稳**
- 磁盘预留 **20GB 以上**

### ④ 网络
```bash
docker pull hello-world   # 先测试镜像链路
```

### ⑤ API Key
如果使用 Chat 功能，需要去 https://sim.ai Settings → Chat keys 生成 `COPILOT_API_KEY`。
Chat 不可用不一定是部署失败，也可能是缺 Key。

## ⚡ 第一关：Docker Compose 最小启动

```bash
git clone https://github.com/simstudioai/sim.git && cd sim
docker compose -f docker-compose.prod.yml up -d
```

建议拆开执行：先 clone，加 `.env` 配置，再启动。

**默认 compose 会启动：**
- Sim 主应用（核心控制台）
- PostgreSQL（数据持久化）
- 其他依赖服务

## 📌 第二关：确认入口可用

启动后访问 `http://localhost:3000`。看到控制台界面表示基础部署成功。

**关键判断：**
- 页面能加载 = Docker 启动正常
- Chat 功能异常 = 缺 API Key（非部署失败）
- 模型无输出 = 模型配置问题

## 🔗 第三关：接入本地模型（Ollama）

Sim 支持连接外部 LLM 提供者。推荐用 Ollama 接入本地模型：

1. 确认 Ollama 运行中
2. 在 Sim 控制台 Settings → Providers 添加 Ollama 端点（默认 `http://host.docker.internal:11434`）
3. 拉一个轻量模型测试，如 `qwen2:1.5b` 或 `llama3.2:3b`

**接入后检查：** 在 Chat 中选择该模型，发送简单问题验证输出。

## 🧪 最小工作场景：研究资料问答台

1. 新建 Knowledge Base，上传 PDF/文档
2. 新建 Agent，绑定知识库
3. 在 Chat 中提问，验证是否基于自己的资料回答
4. 尝试 Workflow：查资料 → 生成摘要 → 写入记录

## 🧯 常见错误

| 问题 | 检查方向 |
|------|---------|
| Docker 没启动 | `docker info` 检查 |
| 端口冲突 | `ss -lntp` 查 3000/3002/5432 |
| 内存不足 | Docker 日志 OutOfMemory 错误，加内存或减服务 |
| Chat 不可用 | 是否已设 `COPILOT_API_KEY` |
| 外部 Ollama 连不上 | 网络模式、host.docker.internal 是否可用 |

## 🧹 回滚

```bash
docker compose -f docker-compose.prod.yml down -v
docker system prune -a   # 清理镜像和容器
```

## 🚀 下一步扩展

- 连接团队数据源（Google Drive、Notion 等）
- 配置可视化工作流编排
- 接入外部 API 和 Webhook
- 多 Agent 协作场景

## ⚖️ 和其他工具对比

| 工具 | 定位 |
|------|------|
| **Sim** | All-in-one Agent workspace，聊/文件/知识库/表格/工作流一体 |
| Open WebUI | 侧重模型聊天和简单 RAG |
| Dify | 偏向低代码 Agent 开发平台 |
| n8n | 偏向工作流自动化，弱 AI Agent |
