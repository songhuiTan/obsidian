# 一天94次提交！我用OpenClaw指挥Claude Code，3天搭出一套AI开发团队

## 文章信息
- **来源**: 公众号"刘邦学AI"
- **链接**: https://mp.weixin.qq.com/s/nTwwhLeFD1g1LH2tZL62uw
- **发布时间**: 2026-03-14 11:43:40
- **作者**: 刘邦

## 核心观点
**OpenClaw不是用来写代码的，它是用来指挥AI写代码的**。它和Claude Code、Codex的关系，就像是将军和士兵——将军负责调度，士兵负责冲锋。

## 环境搭建三步曲

### 第一步：安装 OpenClaw
访问官网安装，按照官方文档操作。

### 第二步：安装 acpx 插件（关键！）
这是 OpenClaw 和 Claude Code/Codex 之间的"通讯器"。

```bash
openclaw plugins install acpx
openclaw config set plugins.entries.acpx.enabled true
openclaw config set acp.enabled true
openclaw config set acp.backend acpx
openclaw config set acp.allowedAgents '["claude","codex","gemini","pi"]'
openclaw gateway restart
```

### 第三步：配置 API Key
```bash
export ANTHROPIC_API_KEY=你的AnthropicKey
export OPENAI_API_KEY=你的OpenAIKey
```

## 三种使用场景

### 场景1：一次性任务（快速搞定）
适用于"干完就走"的任务，如重构代码。

```bash
/acp spawn claude --mode oneshot --thread off
```

### 场景2：持续会话（深度协作）
适用于复杂项目，需要多轮沟通。

```bash
/acp spawn claude --mode persistent --thread auto
```

### 场景3：最省心的方式（推荐）
直接自然语言描述需求，OpenClaw 自动识别并调用 Claude Code。

**示例**："用claude code帮我写一个爬虫，抓取知乎热榜标题"

## Claude Code vs Codex 对比

| 维度 | Claude Code | Codex |
|------|-------------|-------|
| **代码质量** | 非常高，擅长架构设计 | 也很高，偏向快速实现 |
| **上下文理解** | 200K tokens，大项目也能hold住 | 上下文稍短，适合中小项目 |
| **交互体验** | 像和一个资深工程师对话 | 像和一个高效执行者对话 |
| **价格** | Anthropic API按量付费 | OpenAI API按量付费 |

**选择建议**：
- 做架构设计、重构老代码 → 用 Claude Code
- 写脚本、快速验证想法 → 用 Codex
- 两个都装上，看心情换着用

## 真实效果数据

**作者实测项目**：自动化工作流
- 自动生成公众号排版模板
- 自动抓取热点文章并分析
- 自动整理素材到指定文件夹

**成果**：
- Claude Code 帮写了大概 2000 行代码
- 作者只改了不到 100 行
- 从需求到上线，**只用了 3 天**（换以前需要两周）
- 其间还正常处理日常工作、开了 3 个客户会议

**核心价值**：OpenClaw 不是替代你，而是把你的时间放大。

## 常见问题解答

### Q1：API Key 花钱吗？
花。重度使用一个月大概 100-200 块钱。但相比于省下来的时间，这个钱值。

### Q2：公司代码能放上去吗？
不建议。虽然 Anthropic 和 OpenAI 都承诺不用数据训练模型，但敏感代码还是建议在本地跑，或者用企业版。

### Q3：和 Cursor 比，哪个更好？
不是同一个赛道：
- Cursor 是 IDE 插件，适合日常写代码
- Claude Code 是独立 AI 助手，适合做复杂任务、自动化工作流
- 建议两个都用

## 核心洞察

> "OpenClaw就是那个中军帐，Claude Code和Codex就是你的两员大将。"

**AI 不是来抢饭碗的，它是来帮你放大能力的**。

你以前一个人能干一个项目的活，现在有了 OpenClaw + Claude Code/Codex，你能同时干三个项目。而且质量不降反升，因为 AI 不知疲倦、不会犯低级错误。

当然，前提是你要学会指挥它。就像古代名将指挥千军万马，你不会因为士兵多就赢，你得会排兵布阵。

## 参考资料

- OpenClaw 官方文档：docs.openclaw.ai
- Claude Code 官方指南：docs.anthropic.com
- Codex 官方文档：openai.com/codex

---

**归档时间**: 2026-03-17 17:45
**归档人**: 小龙虾 🦞
**标签**: #OpenClaw #ClaudeCode #Codex #AI编程 #效率工具
