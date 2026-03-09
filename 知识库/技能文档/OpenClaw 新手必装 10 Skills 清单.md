# OpenClaw 新手必装 10 Skills 清单

> 整理自飞书文档：OpenClaw 新手必装 10 Skills 安装使用教程

## 📦 一键安装命令

```bash
npx skills add useai-pro/openclaw-skills-security@skill-vetter -g -y
npx skills add charon-fan/agent-playbook@self-improving-agent -g -y
npx skills add kepano/obsidian-skills@obsidian-cli -g -y
npx skills add alphaonedev/openclaw-graph@playwright-scraper -g -y
npx skills add composiohq/awesome-claude-skills@github-automation -g -y
npx skills add shipshitdev/library@execution-accelerator -g -y
npx skills add agentmail-to/agentmail-skills@agentmail -g -y
npx skills add openai/skills@notion-research-documentation -g -y
npx skills add openclaudia/openclaudia-skills@feishu-lark -g -y
npx skills add travisjneuman/.claude@macos-native -g -y
npx skills add sundial-org/awesome-openclaw-skills@clawdbot-backup -g -y
```

## 🎯 安装前准备

1. 本机已安装 Node.js（建议 18+）
2. 可使用 `npx` 命令
3. 推荐安装方式：`npx skills add <owner/repo@skill> -g -y`
   - `-g` 表示全局安装（用户级）
   - `-y` 表示跳过交互确认

## 📋 逐个技能详解

### 1. skill-vetter（先体检再安装）

**是什么**：技能安装前的安检层

**解决问题**：
- 恶意脚本
- 命令注入
- 脏提示词

**不装后果**：把机器权限直接暴露给陌生代码

**Skills 地址**：https://skills.sh/useai-pro/openclaw-skills-security/skill-vetter

**安装命令**：`npx skills add useai-pro/openclaw-skills-security@skill-vetter -g -y`

**OpenClaw 手动用法**：先用 skill-vetter 检查这个新技能仓库，再告诉我是否可装。

**自动触发时机**（若开启）：你提到"安装新 skill / 安全检查 / 风险审计"时。

---

### 2. Self-Improving Agent（自动复盘）

**是什么**：能记录错误和偏好的自改进代理

**解决问题**：
- 同类问题重复出现
- 每次都要重新教

**不装后果**：持续在同一类坑里重复返工

**Skills 地址**：https://skills.sh/charon-fan/agent-playbook/self-improving-agent

**安装命令**：`npx skills add charon-fan/agent-playbook@self-improving-agent -g -y`

**OpenClaw 手动用法**：把这次任务的失败点做成复盘卡，写入下次执行规范。

**自动触发时机**（若开启）：任务结束、报错修复后，且你提到"复盘/总结/下次避免"时。

---

### 3. Obsidian Direct（知识沉淀）

**是什么**：把过程和结果沉淀到 Obsidian 的技能链路

**解决问题**：
- 信息散
- 找不到
- 无法复用

**不装后果**：下次还要从零搜、从零做

**Skills 地址**：https://skills.sh/kepano/obsidian-skills/obsidian-cli

**安装命令**：`npx skills add kepano/obsidian-skills@obsidian-cli -g -y`

**OpenClaw 手动用法**：把今天这个方案整理成 Obsidian 笔记，按项目和主题建双向链接。

**自动触发时机**（若开启）：你提到"归档到知识库/沉淀笔记/写入 Obsidian"时。

---

### 4. Playwright Scraper（网页自动抓数）

**是什么**：模拟真人操作网页并自动提取数据

**解决问题**：
- 手工复制粘贴慢
- 容易出错

**不装后果**：大量时间耗在机械搬运上

**Skills 地址**：https://skills.sh/alphaonedev/openclaw-graph/playwright-scraper

**安装命令**：`npx skills add alphaonedev/openclaw-graph@playwright-scraper -g -y`

**OpenClaw 手动用法**：用 playwright-scraper 抓这个页面的表格，导出 CSV 并给我汇总。

**自动触发时机**（若开启）：你提到"抓网页/批量采集/导出表格"时。

**实战记录**：
- ✅ 2026-03-09 成功安装并使用
- ✅ 完整下载 Chromium 浏览器（340 MB）
- ✅ 编写自动化抓取脚本
- ⚠️ 飞书文档动态渲染复杂，需要进一步优化

---

### 5. Git 组合包（Review + Issue + 交付）

**是什么**：围绕 GitHub/Git 工作流自动化的一组能力

**解决问题**：Review、Issue、交付流程断裂

**不装后果**：项目忙起来就进入救火模式

**Skills 地址**：https://skills.sh/composiohq/awesome-claude-skills/github-automation

**安装命令**：`npx skills add composiohq/awesome-claude-skills@github-automation -g -y`

**OpenClaw 手动用法**：用 github-automation 按这份需求拆 Issue，并给每个 Issue 补验收标准。

**自动触发时机**（若开启）：你提到"开 Issue / 建 PR / 跑 review 流程"时。

---

### 6. Take the Wheel（执行加速）

**是什么**：把任务拆成下一步动作并推动执行

**解决问题**：想得多、开工慢

**不装后果**：计划很满但产出很少

**Skills 地址**：https://skills.sh/shipshitdev/library/execution-accelerator

**安装命令**：`npx skills add shipshitdev/library@execution-accelerator -g -y`

**OpenClaw 手动用法**：把这个目标拆成今天可执行的 3 步，并直接开始第一步。

**自动触发时机**（若开启）：你提到"开工/下一步/执行计划"但目标较大时。

---

### 7. AgentMail Integration（身份隔离）

**是什么**：多任务多邮箱身份管理

**解决问题**：账号混用、权限串线、通知污染

**不装后果**：一个链路出问题会牵连全局

**Skills 地址**：https://skills.sh/agentmail-to/agentmail-skills/agentmail

**安装命令**：`npx skills add agentmail-to/agentmail-skills@agentmail -g -y`

**OpenClaw 手动用法**：给这个项目创建独立 AgentMail 身份，并配置收发规则。

**自动触发时机**（若开启）：你提到"创建项目邮箱/多身份/邮件自动化"时。

---

### 8. Notion / 飞书插件（协同上云）

**是什么**：把结果自动同步到团队协作系统

**解决问题**：本地有结果，但团队看不到

**不装后果**：信息断层，反复催进度

**Notion 地址**：https://skills.sh/openai/skills/notion-research-documentation

**飞书地址**：https://skills.sh/openclaudia/openclaudia-skills/feishu-lark

**安装命令**：
```bash
npx skills add openai/skills@notion-research-documentation -g -y
npx skills add openclaudia/openclaudia-skills@feishu-lark -g -y
```

**OpenClaw 手动用法**：把这份周报同步到 Notion，再把摘要发到飞书群。

**自动触发时机**（若开启）：你提到"同步 Notion/发飞书/更新协作文档"时。

---

### 9. Mac Native Suite（系统层联动）

**是什么**：把 Agent 接到 macOS 原生能力

**解决问题**：建议很多，但执行靠手点

**不装后果**：自动化停在口头阶段

**Skills 地址**：https://skills.sh/travisjneuman/.claude/macos-native

**安装命令**：`npx skills add travisjneuman/.claude@macos-native -g -y`

**OpenClaw 手动用法**：把这个待办写入日历，明天 9:00 提醒我，并附执行链接。

**自动触发时机**（若开启）：你提到"创建提醒/写入日历/系统通知"时。

---

### 10. Cron Backup（容灾备份）

**是什么**：定时备份配置与关键上下文

**解决问题**：误删、崩溃、换机后恢复困难

**不装后果**：一次事故就可能重头来过

**Skills 地址**：https://skills.sh/sundial-org/awesome-openclaw-skills/clawdbot-backup

**安装命令**：`npx skills add sundial-org/awesome-openclaw-skills@clawdbot-backup -g -y`

**OpenClaw 手动用法**：现在执行一次全量备份，并生成恢复演练清单。

**自动触发时机**（建议）：每天凌晨 02:30 定时触发；每次重大改动后再手动补一次。

---

## 🎮 使用建议

### 手动触发优先
在指令里直接点名技能（最稳）。

### 自动触发可选
依赖技能的 description/router 配置。若未自动触发，就手动点名。

## 📚 文档结构
- 0) 安装前准备
- 1) 一键安装清单（可直接复制）
- 2) 逐个技能怎么用
- 3) 推荐安装顺序（和视频一致）
- 4) 安装后自检
- 5) 给新手的一句结论

---

## 🎬 推荐安装顺序（和视频一致）

1. skill-vetter
2. self-improving-agent
3. obsidian-cli
4. playwright-scraper
5. github-automation
6. execution-accelerator
7. agentmail
8. notion-research-documentation + feishu-lark
9. macos-native
10. clawdbot-backup

**安装建议**：按顺序逐个安装，每安装一个就测试基本功能，确保无问题后再继续下一个。

---

## 🔍 安装后自检

```bash
# 检查已安装的技能
npx skills check

# 更新已安装的技能
npx skills update
```

**可选操作**：
- 重新搜索替代技能：`npx skills find <关键词>`
- 浏览技能目录：https://skills.sh/

---

## 💡 给新手的一句结论

**先把这 10 个装齐，再去追求复杂玩法。这样做的价值不是"看起来高级"，而是减少返工、提高稳定性、让结果可复用。**

## 🔗 相关链接
- 飞书原文档：https://lcn32lxqhc07.feishu.cn/wiki/QsSAwtXibiowgokAVXhcuupAnMe
- skills.sh 官网：https://skills.sh/

## 📌 标签
#OpenClaw #技能清单 #必装技能 #新手入门

---
**创建时间**：2026-03-09
**最后更新**：2026-03-09
**作者**：小龙虾 🦞
