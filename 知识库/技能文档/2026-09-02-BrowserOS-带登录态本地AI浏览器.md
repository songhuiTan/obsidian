---
title: "Claude Code / Cursor 的完美浏览器搭子：免风控、带登录态、全本地运行"
source: "知识姬 Mina（知识发电机）"
source_url: "https://mp.weixin.qq.com/s/E--OWI9CXDJ_ae3G_3WXEA"
date: "2026-09-02"
tags: [BrowserOS, AI浏览器, MCP, 网页自动化, 登录态, 本地优先, BYOK, 开源]
---

# BrowserOS：带真实登录态、免风控的本地 AI 浏览器

> **一句话**：1.3 万 Star 开源项目。核心洞察：AI 网页自动化最大瓶颈不是模型智商，而是**缺乏一个靠谱的、带真实登录态的、本地稳定运行的浏览器环境**。解法：本地拆成两个浏览器——一个给 AI 跑腿（BrowserOS neo），一个留给人用（BrowserOS 主力浏览器）。

## 双浏览器架构

**1. BrowserOS neo：专供 AI 跑腿的二号浏览器**
- 不是给人日常冲浪用的，是 Agent 的工作间
- 一键克隆 Chrome 的登录状态、书签、插件
- 通过 MCP 接入 Claude Code、Codex、Cursor、VS Code、Zed
- AI 在专属浏览器开独立标签页干活，人在主浏览器继续工作，互不干扰

**2. BrowserOS：带本地原生 Agent 的日常主力浏览器**
- 深度定制的 Chromium 分支，自动化工具链内嵌底层引擎
- 随时唤出侧边栏对话、挂载定时任务
- 一键接入本地 Ollama、LM Studio，告别云端窥探

## 解决传统自动化的三大硬伤

### 1. 登录态隔离与风控（核心）
- **传统方案的问题**：Playwright 默认启动干净沙盒，每次要人肉过双重验证；Browserbase 等云浏览器 IP 在数据中心——LinkedIn/Twitter/企业内网几乎"见光死"
- **BrowserOS neo 的做法**：直接跑在本机 `127.0.0.1`，挂着日常家庭/公司网络，带真实登录 Cookie——对目标网站就是一次普通日常访问，**风控误杀率断崖式下降**

### 2. 操作复盘录像
- AI 每个操作步骤完整录制成**可视化时间轴视频**，存本地磁盘
- 可拖动进度条一帧帧回放：点了哪个按钮、填了哪些表单
- 发现逻辑偏差直接叫停纠正（"像看监控录像一样复盘"）

### 3. 本地优先 BYOK（自带密钥）
- 用 Claude/OpenAI/Gemini 填自己的 API Key；注重隐私切本地开源模型
- Session/截图/日志全锁本地 `~/.browserclaw/`
- 官方只保留基础匿名连接统计，设置里一键关闭

## 技术架构（极客味工程）

- **底层**：Chromium 深度裁剪与打补丁的构建分支（融入部分 ungoogled-chromium 隐私补丁），保留 Manifest V2 广告拦截插件（uBlock Origin）原生支持
- **上层**：Rust（高性能 MCP 协议端点 + 本地 API）、TypeScript、Go 混合；Bun 服务端处理 Agent 决策循环；前端 WXT + React 轻量面板
- **关键优势**：高频 CDP（Chrome DevTools Protocol）交互极轻量，比 Chrome 扩展反复截长图喂视觉模型的方案 **token 消耗精简得多**

## 落地场景

- **社交媒体多账号维护**：定时抓排期文档 → 打开后台自动配图/排版/发布 → 统计互动数据
- **无 API 系统的自动化**：老 CRM、财务报销系统无开放接口 → 大白话下指令批量表单录入
- **深层内容采集**：登录才能看的研报、社区帖 → 本地 Agent 打开阅读、提炼观点、结构化存本地

## 战略分析

**1. 与本机 Hermes 的关系（直接相关）：** Hermes 自带 browser 工具集就是本地浏览器（stealth_features: local），今天刚经历的微信验证码事件就是活例子——firecrawl 云端代理被微信风控拦截（CAPTCHA），而 Hermes 本地浏览器能通过。BrowserOS 的思路进一步：**带真实 Cookie + 真实网络出口的浏览器**，对需要登录态的场景（后台/研报/企业系统）是 Hermes browser 工具的强化方向。

**2. 与最近归档谱系的关联：**

| 维度 | BrowserOS | 08-09 AI GUI Agent 对比 | TokBrain | Hermes browser |
|---|---|---|---|---|
| 形态 | 本地双浏览器 + MCP | 框架/模型（Browser Use/UI-TARS 等） | 抖音采集产品 | 内置无头/本地浏览器 |
| 登录态 | ✅ 克隆真实 Cookie | 部分支持 | 权限感知 | ❌ 无 Cookie |
| 风控 | ✅ 真实网络出口 | 各有方案 | - | 本地可过部分 |
| 复盘 | ✅ 操作录像时间轴 | - | - | 无 |
| 成本 | CDP 轻量省 token | 视觉模型高 token | - | - |

**3. 可借鉴的三个设计：**
- **双浏览器人机分离**：AI 专用浏览器 + 用户主浏览器互不干扰——解决"人机混用打架"（对应桌面自动化四项目对比里的 Browser Use 思路）
- **操作录像复盘**："AI 干了什么可回放"——与 AgentDesk"Git 证据"、J-Space"强制验证"同哲学：**可审计性是 AI 执行信任的基础**
- **BYOK + 本地日志**：密钥自持、数据锁本地 `~/.browserclaw/`——与用户"数据不出域"偏好一致

**4. 落地建议：** 如果用户常需要"登录态网页操作"（公众号后台、后台数据抓取、多账号社交维护），BrowserOS neo 值得装来配 Hermes 用（MCP 接入）；若只是普通网页读取，Hermes 内置 browser + firecrawl 已够用。注意：macOS 支持情况需查官方仓库（文章主打 Chromium 桌面端）。

## 关联阅读

- [[2026-08-09-桌面自动化探究-AI-GUI-Agent四项目对比]] — Browser Use/UI-TARS/OmniParser/cua 对比，BrowserOS 是浏览器形态补充
- [[2026-08-18-TokBrain-抖音内容本地多模态知识库]] — 登录态采集场景的另一种方案
- [[2026-08-18-AgentDesk-任务卡驱动多Agent开发工作流]] — 可审计性哲学（操作录像 vs Git 证据）

## 归档日志

- 2026-09-02 归档（Firecrawl 成功；封面 + MCP 接入图已存 assets）
