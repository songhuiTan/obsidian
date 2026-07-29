# 浏览器内 GUI Agent 全景：Page Agent、browser-use、Stagehand、Skyvern 技术路线解析

> **来源**：最新AI知识宇宙（andrew003）  
> **日期**：2026-07-03  
> **原文链接**：https://mp.weixin.qq.com/s/T5vSdwGy1gTv0PdUaLj7OA  
> **分类**：AI Agent 框架与工程化

---

## 文章概要

横向拆解浏览器 GUI Agent 五条技术路线的架构差异与选型逻辑。核心分野围绕三个问题：**Agent 站在哪里？用什么方式"看"页面？如何执行动作？**

---

## 五条技术路线全景

### 路线一：外部 CDP 驱动（Playwright / Puppeteer）
- **Playwright**（微软）：跨浏览器、内置自动等待、MCP 支持
- **Puppeteer**（Google）：轻量、Chromium/Edge 导向
- 定位：通用自动化与测试底座，上层 Agent 框架的"底层执行器"

### 路线二：Agent 优先的结构化库（browser-use / Stagehand）
- **browser-use**（Python）：Agent 优先，观察→决定→执行→评估推理循环，读取 DOM + 无障碍树，$17M 种子轮
- **Stagehand**（Browserbase）：JS SDK，`act`/`extract`/`observe` 三原子原语，22k⭐，Agent 自主+传统脚本混合
- 共同点：DOM/无障碍树为主要感知，确定性和可读性

### 路线三：视觉 / 计算机使用（Skyvern / OpenAI Operator）
- **Skyvern**：视觉优先，截图+DOM，抗布局变化
- **OpenAI Operator（CUA）**：GPT-4o 视觉 + 强化学习，2026 演进为 ChatGPT Agent
- 取舍：通用性换性能，能处理 Canvas/视频但慢且贵

### 路线四：AI 优先紧凑 CLI（Vercel agent-browser）
- 100% 原生 Rust CLI，紧凑无障碍树快照 + 短引用 token（`@e3`、`@e7`）
- 相比 JSON/完整 DOM 节省高达 **93% 上下文窗口**
- 定位：给 AI 省 token、省钱、省延迟

### 路线五：页内轻量嵌入（阿里 Page Agent）← 本文主角
- **DOM 脱水（Dehydration）**：将实时 DOM 压缩为 FlatDomTree 纯文本映射
- **零基础设施**：无需扩展、Python、无头浏览器
- **纯文本，不用截图**：不需要多模态 LLM
- **一行集成**：`<script>` 标签或 `npm install page-agent`
- MIT 许可，致谢 browser-use
- GitHub 20k+⭐

```javascript
import { PageAgent } from 'page-agent'
const agent = new PageAgent({
  model: 'qwen3.5-plus',
  baseURL: 'https://dashscope.aliyuncs.com/compatible-mode/v1',
  apiKey: 'YOUR_API_KEY',
  language: 'zh-CN',
})
await agent.execute('点击登录按钮，然后把用户名填成 John')
```

---

## 横向对比

| 方案 | 运行位置 | 感知方式 | 核心定位 | 许可 |
|:---|:---|:---|:---|:---|
| **Playwright/Puppeteer** | 外部进程 | DOM / CDP | 通用自动化+测试底座 | Apache/BSD |
| **browser-use** | 外部进程 | DOM+无障碍树 | Agent 优先自动化 | MIT |
| **Stagehand** | 外部进程 | DOM+可选视觉 | 原子操作+Agent混合 | MIT |
| **Skyvern/Operator** | 外部/云 | 截图视觉为主 | 抗布局变化视觉Agent | AGPL/闭源 |
| **Vercel agent-browser** | 外部进程 | 无障碍树+ref token | 为AI省token | 开源 |
| **阿里 Page Agent** | 浏览器内 | FlatDomTree 文本 | Web应用嵌入副驾 | MIT |

---

## 三个关键维度

| 维度 | 分野 |
|:---|:---|
| **站位** | 外部（CDP拉线） vs 页内（零基建、继承会话态） |
| **感知** | DOM 文本（快/精确/便宜，但 Canvas/shadow DOM 失灵） vs 视觉截图（通用慢贵） |
| **执行** | CDP 发指令 vs 页内原生 DOM 事件 |

> 2026 年生产栈已收敛到 **hybrid**：DOM 为主、视觉兜底。

---

## Benchmark 真相

- WebVoyager 基准各家 90%+（偏读取、仅 643 任务/15 网站）
- 真实动态环境（Online-Mind2Web）成功率大幅崩塌，仅 Operator 约 61%
- **选型要在目标网页上实测，不要只看厂商 benchmark**

---

## 选型指南

| 需求 | 推荐方案 |
|:---|:---|
| 给 Web 应用加 AI 副驾 | **Page Agent**（零基建一行集成） |
| 服务端自主多步 Agent | **browser-use / Stagehand** |
| 动态渲染复杂界面 | **Skyvern / Operator**（视觉兜底） |
| 省 token 的 CLI Agent | **Vercel agent-browser** |
| 纯测试与确定性自动化 | **Playwright / Puppeteer** |

---

## 行业意义：三条路之争

1. **外部控制派** — Agent 在外，浏览器是被操控对象。成熟可批量
2. **视觉识别派** — Agent 像人一样看屏幕。最通用最贵
3. **页内嵌入派** — Agent 住进页面。门槛最低、边界最窄

Page Agent 真正价值：把"给每个 Web 应用装 AI 副驾"的成本降到几乎为零。

---

## 关键词

`GUI Agent` `Page Agent` `browser-use` `Stagehand` `Skyvern` `OpenAI Operator` `Vercel agent-browser` `Playwright` `Puppeteer` `DOM 脱水` `FlatDomTree` `CDP` `浏览器自动化` `AI 副驾`
