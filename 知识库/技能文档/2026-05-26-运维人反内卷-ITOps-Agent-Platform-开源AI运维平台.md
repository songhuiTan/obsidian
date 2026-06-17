---
title: "运维人反内卷：我写了个 AI 运维平台，2 分钟部署，现在开源了"
source: "谭策"
source_url: "https://mp.weixin.qq.com/s/NDqYrfqR0RZEvSESyVD2hg"
date: "2026-05-26"
tags: [AIOps, ITOps, 开源, 运维自动化, Agent 平台, 部署]
---

> 这是个人项目，代码写得不够优雅，文档也不够完善，但它是真心实意想要解决运维人的痛点。如果你也经历过那些熬夜排障的夜晚，也许你会理解我想做的事情。

**项目地址：**
- GitHub：https://github.com/qinshihu/itops-agent-platform
- Gitee：https://gitee.com/IT_Oline/itops-agent-platform
- GitCode：https://gitcode.com/gcw_IM7aAihp/itops-agent-platform

![封面图](../assets/2026-05-26-ITOps-Agent-Platform/cover.jpg)

## 引言

你有没有过这样的夜晚？凌晨 2 点，手机在枕头底下疯狂震动，是 Zabbix 的告警短信。你从床上弹起来，眼睛都睁不开，摸过电脑就开始 SSH 登录服务器。翻 3 个 G 的日志，查 CPU、内存、磁盘、进程，杀僵尸进程，重启服务，折腾到 4 点终于解决。第二天早上 9 点，你还要准时到公司，写故障报告，复盘，整改，优化监控。日复一日，年复一年。

我一直在想：**这些重复性的工作，能不能让 AI 来帮忙？**

于是有了这个项目——**ITOps Agent Platform**，一个我利用业余时间开发的运维自动化平台。它的核心想法很简单：把运维经验变成 AI Agent 的能力，让机器去巡检、去诊断、去处理告警，让人去做更有价值的事情。

## 一、ITOps Agent Platform 是什么？

ITOps Agent Platform 是一个基于大语言模型的企业级运维自动化平台，通过可视化工作流编排多个 AI Agent 协同工作，实现服务器巡检、告警处理、故障诊断、合规检查等运维任务的自动化。

### 核心特性

| 特性 | 说明 |
| --- | --- |
| 🤖 多 Agent 协作 | 9 个预设运维 Agent，覆盖告警、诊断、巡检、变更等场景 |
| 🔀 可视化工作流 | 拖拽式编排，支持串行/并行/条件分支 |
| 💻 Web SSH 终端 | 基于 xterm.js 的交互式远程终端 |
| 🖥️ 主机管理 | 多级分组树形结构、Excel 批量导入、SSH 自动信息采集 |
| 🔔 告警中心 | Webhook 接收 Prometheus/Zabbix/通用告警，自动降噪 |
| 📚 知识库 + RAG | 智能检索注入 LLM 上下文 |
| 🗣️ AI Copilot | 自然语言对话式运维助手 |
| 📈 数据大屏 | 实时运维数据可视化监控，支持投屏展示 |
| 🧠 支持本地 AI 模型 | 兼容 Ollama 等本地部署的大模型，数据不出域 |
| 🔒 企业级安全 | AES-256-GCM 加密、JWT 认证、速率限制、审计日志 |

## 二、部署方式一：一键部署（推荐）

运行下面一条命令即可：

```bash
curl -sL https://gitee.com/IT_Oline/itops-agent-platform/raw/main/deploy.sh -o deploy.sh && chmod +x deploy.sh && ./deploy.sh
```

脚本支持 `-y` 参数自动确认提示（`./deploy.sh -y`），也可手动输入 `y` 确认。

☕ 喝杯咖啡的功夫（2 分钟）平台就跑起来了，脚本会自动完成：

1. 检查 Docker 环境
2. 从阿里云拉取最新镜像
3. 生成随机 JWT_SECRET
4. 启动前后端服务
5. 验证健康状态

部署完成约 1-3 分钟，看到以下输出表示成功：

```
前端: http://你的服务器IP:8080
健康检查：http://localhost:3001/health
```

## 三、部署方式二：Docker Compose 部署

### 3.1 获取项目代码

```bash
git clone https://github.com/qinshihu/itops-agent-platform.git
cd itops-agent-platform
```

### 3.2 配置环境变量

```bash
cp .env.example .env
vim .env
```

编辑 `.env`，至少设置 `JWT_SECRET`（可用 `openssl rand -hex 32` 生成）：

```bash
JWT_SECRET=your-random-secure-secret-key-here
```

> **重要**：AI API 密钥（`DOUBAO_API_KEY` / `OPENAI_API_KEY`）不需要在 `.env` 中配置！AI 密钥可在登录后通过前端页面配置，保存到数据库，重启不丢失。

### 3.3 启动服务

```bash
# 拉取镜像并启动服务
docker compose up -d

# 查看服务状态
docker compose ps

# 查看后端日志
docker compose logs -f backend

# 查看前端日志
docker compose logs -f frontend
```

### 3.4 验证部署

```bash
# 检查后端健康状态
curl http://localhost:3001/health

# 检查前端是否可访问
curl -I http://localhost:8080
```

## 四、访问和使用

### 4.1 登录系统

浏览器访问：`http://<服务器IP>:8080`

### 4.2 默认管理员账号

| 项目 | 值 |
| --- | --- |
| 用户名 | admin |
| 密码 | admin |

> ⚠️ 安全提示：首次登录后系统会强制要求修改密码。

### 4.3 配置 AI API 密钥

登录后，进入 **设置 → AI 配置**，填写以下信息：

| 配置项 | 说明 |
| --- | --- |
| 豆包 API（国内用户） | 在火山引擎控制台获取密钥 |
| OpenAI API | 在 OpenAI 平台获取密钥 |
| 模型 ID | 如 `doubao-pro-32k`、`gpt-4o` 等 |

配置后立即可用，所有 AI Agent、Copilot、RAG 功能将自动启用。

## 五、系统要求

| 项目 | 最低配置 | 推荐配置 |
| --- | --- | --- |
| CPU | 1 核 | 2 核及以上 |
| 内存 | 1 GB | 2 GB 及以上 |
| 磁盘 | 20 GB | 20 GB 及以上 |
| 系统 | CentOS 7+ / AlmaLinux 10 | 最新版 |

## 六、服务管理

```bash
# 启动
docker compose up -d

# 停止
docker compose down

# 重启
docker compose restart

# 查看实时日志
docker compose logs -f

# 更新
docker compose pull
docker compose up -d --build
```

## 七、一起把这件事做成

先说实话：我一个人做不完。这个项目从我有想法到现在，已经写了大半年。后端架构设计得不够好，前端代码写得很"运维风"，测试覆盖低得可怜，文档更是惨不忍睹。

但我相信，这个项目值得被做得更好。因为它解决的，是真真切切的每一个运维人的痛点。

> 一个人可以走得很快，但一群人可以走得很远。

**你能在这里做什么？**

- 🟢 **零门槛**：点个 Star、转发、提 Issue、改错别字
- 🟡 **入门级**：整理文档、写部署教程、优化前端样式
- 🟠 **进阶级**：修复 Bug、写 Agent 插件（如 MySQL 巡检）
- 🔴 **挑战级**：把 SQLite 换成 PostgreSQL、重构路由模块
- ⏫ **AI 编程工具使用者**：用 Cursor/Copilot 辅助写测试、重构代码、翻译文档

**GitHub：** https://github.com/qinshihu/itops-agent-platform
**Gitee：** https://gitee.com/IT_Oline/itops-agent-platform
**GitCode：** https://gitcode.com/gcw_IM7aAihp/itops-agent-platform
