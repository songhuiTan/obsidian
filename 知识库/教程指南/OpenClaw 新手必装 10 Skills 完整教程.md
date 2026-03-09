# OpenClaw 新手必装 10 Skills 完整教程

> 文档来源：飞书云文档 | 整理者：抖音@睡觉的大提琴 | 学习日期：2026-03-09

---

## 📋 目录
1. [安装前准备](#0-安装前准备)
2. [一键安装清单](#1-一键安装清单可直接复制)
3. [逐个技能详解](#2-逐个技能详解)
4. [推荐安装顺序](#3-推荐安装顺序和视频一致)
5. [安装后自检](#4-安装后自检)
6. [给新手的建议](#5-给新手的一句结论)

---

## 0) 安装前准备

### 前置条件
1. ✅ 本机已安装 Node.js（建议 18+）
2. ✅ 可使用 `npx` 命令
3. ✅ 推荐安装方式（统一）：

```bash
npx skills add <owner/repo@skill> -g -y
```

### 参数说明
- `-g`：全局安装（用户级）
- `-y`：跳过交互确认

### 技能映射
本教程地址优先使用 https://skills.sh/，并按"视频概念 → skills.sh 对应技能"做映射。

---

## 1) 一键安装清单（可直接复制）

```bash
# 1. skill-vetter（先体检再安装）
npx skills add useai-pro/openclaw-skills-security@skill-vetter -g -y

# 2. Self-Improving Agent（自动复盘）
npx skills add charon-fan/agent-playbook@self-improving-agent -g -y

# 3. Obsidian Direct（知识沉淀）
npx skills add kepano/obsidian-skills@obsidian-cli -g -y

# 4. Playwright Scraper（网页自动抓数）
npx skills add alphaonedev/openclaw-graph@playwright-scraper -g -y

# 5. Git 组合包（Review + Issue + 交付）
npx skills add composiohq/awesome-claude-skills@github-automation -g -y

# 6. Take the Wheel（执行加速）
npx skills add shipshitdev/library@execution-accelerator -g -y

# 7. AgentMail Integration（身份隔离）
npx skills add agentmail-to/agentmail-skills@agentmail -g -y

# 8. Notion / 飞书插件（协同上云）
npx skills add openai/skills@notion-research-documentation -g -y
npx skills add openclaudia/openclaudia-skills@feishu-lark -g -y

# 9. Mac Native Suite（系统层联动）
npx skills add travisjneuman/.claude@macos-native -g -y

# 10. Cron Backup（容灾备份）
npx skills add sundial-org/awesome-openclaw-skills@clawdbot-backup -g -y
```

---

## 2) 逐个技能详解

### 使用建议
**OpenClaw 下建议：**
- 🎯 **手动触发优先**：在指令里直接点名技能（最稳）
- ⚙️ **自动触发可选**：依赖技能的 description/router 配置。若未自动触发，就手动点名

---

### 1. skill-vetter（先体检再安装）

#### 是什么
技能安装前的安检层

#### 解决问题
- 恶意脚本
- 命令注入
- 脏提示词

#### 不装后果
把机器权限直接暴露给陌生代码

#### Skills 地址
https://skills.sh/useai-pro/openclaw-skills-security/skill-vetter

#### 安装命令
```bash
npx skills add useai-pro/openclaw-skills-security@skill-vetter -g -y
```

#### OpenClaw 手动用法
先用 skill-vetter 检查这个新技能仓库，再告诉我是否可装。

#### 自动触发时机（若开启）
你提到"安装新 skill / 安全检查 / 风险审计"时。

---

### 2. Self-Improving Agent（自动复盘）

#### 是什么
能记录错误和偏好的自改进代理

#### 解决问题
- 同类问题重复出现
- 每次都要重新教

#### 不装后果
持续在同一类坑里重复返工

#### Skills 地址
https://skills.sh/charon-fan/agent-playbook/self-improving-agent

#### 安装命令
```bash
npx skills add charon-fan/agent-playbook@self-improving-agent -g -y
```

#### OpenClaw 手动用法
把这次任务的失败点做成复盘卡，写入下次执行规范。

#### 自动触发时机（若开启）
任务结束、报错修复后，且你提到"复盘/总结/下次避免"。

---

### 3. Obsidian Direct（知识沉淀）

#### 是什么
把过程和结果沉淀到 Obsidian 的技能链路

#### 解决问题
- 信息散
- 找不到
- 无法复用

#### 不装后果
下次还要从零搜、从零做

#### Skills 地址
https://skills.sh/kepano/obsidian-skills/obsidian-cli

#### 安装命令
```bash
npx skills add kepano/obsidian-skills@obsidian-cli -g -y
```

#### OpenClaw 手动用法
把今天这个方案整理成 Obsidian 笔记，按项目和主题建双向链接。

#### 自动触发时机（若开启）
你提到"归档到知识库/沉淀笔记/写入 Obsidian"时。

---

### 4. Playwright Scraper（网页自动抓数）

#### 是什么
模拟真人操作网页并自动提取数据

#### 解决问题
- 手工复制粘贴慢
- 容易出错

#### 不装后果
大量时间耗在机械搬运上

#### Skills 地址
https://skills.sh/alphaonedev/openclaw-graph/playwright-scraper

#### 安装命令
```bash
npx skills add alphaonedev/openclaw-graph@playwright-scraper -g -y
```

#### OpenClaw 手动用法
用 playwright-scraper 抓这个页面的表格，导出 CSV 并给我汇总。

#### 自动触发时机（若开启）
你提到"抓网页/批量采集/导出表格"时。

#### 实战记录
- ✅ 2026-03-09 成功安装并使用
- ✅ 完整下载 Chromium 浏览器（340 MB）
- ✅ 编写自动化抓取脚本，支持登录认证
- ⚠️ 飞书文档动态渲染复杂，需要进一步优化选择器

---

### 5. Git 组合包（Review + Issue + 交付）

#### 是什么
围绕 GitHub/Git 工作流自动化的一组能力

#### 解决问题
Review、Issue、交付流程断裂

#### 不装后果
项目忙起来就进入救火模式

#### Skills 地址
https://skills.sh/composiohq/awesome-claude-skills/github-automation

#### 安装命令
```bash
npx skills add composiohq/awesome-claude-skills@github-automation -g -y
```

#### OpenClaw 手动用法
用 github-automation 按这份需求拆 Issue，并给每个 Issue 补验收标准。

#### 自动触发时机（若开启）
你提到"开 Issue / 建 PR / 跑 review 流程"时。

---

### 6. Take the Wheel（执行加速）

#### 是什么
把任务拆成下一步动作并推动执行

#### 解决问题
想得多、开工慢

#### 不装后果
计划很满但产出很少

#### Skills 地址
https://skills.sh/shipshitdev/library/execution-accelerator

#### 安装命令
```bash
npx skills add shipshitdev/library@execution-accelerator -g -y
```

#### OpenClaw 手动用法
把这个目标拆成今天可执行的 3 步，并直接开始第一步。

#### 自动触发时机（若开启）
你提到"开工/下一步/执行计划"但目标较大时。

---

### 7. AgentMail Integration（身份隔离）

#### 是什么
多任务多邮箱身份管理

#### 解决问题
- 账号混用
- 权限串线
- 通知污染

#### 不装后果
一个链路出问题会牵连全局

#### Skills 地址
https://skills.sh/agentmail-to/agentmail-skills/agentmail

#### 安装命令
```bash
npx skills add agentmail-to/agentmail-skills@agentmail -g -y
```

#### OpenClaw 手动用法
给这个项目创建独立 AgentMail 身份，并配置收发规则。

#### 自动触发时机（若开启）
你提到"创建项目邮箱/多身份/邮件自动化"时。

---

### 8. Notion / 飞书插件（协同上云）

#### 是什么
把结果自动同步到团队协作系统

#### 解决问题
本地有结果，但团队看不到

#### 不装后果
信息断层，反复催进度

#### Skills 地址
- **Notion**：https://skills.sh/openai/skills/notion-research-documentation
- **飞书**：https://skills.sh/openclaudia/openclaudia-skills/feishu-lark

#### 安装命令
```bash
npx skills add openai/skills@notion-research-documentation -g -y
npx skills add openclaudia/openclaudia-skills@feishu-lark -g -y
```

#### OpenClaw 手动用法
把这份周报同步到 Notion，再把摘要发到飞书群。

#### 自动触发时机（若开启）
你提到"同步 Notion/发飞书/更新协作文档"时。

---

### 9. Mac Native Suite（系统层联动）

#### 是什么
把 Agent 接到 macOS 原生能力

#### 解决问题
建议很多，但执行靠手点

#### 不装后果
自动化停在口头阶段

#### Skills 地址
https://skills.sh/travisjneuman/.claude/macos-native

#### 安装命令
```bash
npx skills add travisjneuman/.claude@macos-native -g -y
```

#### OpenClaw 手动用法
把这个待办写入日历，明天 9:00 提醒我，并附执行链接。

#### 自动触发时机（若开启）
你提到"创建提醒/写入日历/系统通知"时。

---

### 10. Cron Backup（容灾备份）

#### 是什么
定时备份配置与关键上下文

#### 解决问题
- 误删
- 崩溃
- 换机后恢复困难

#### 不装后果
一次事故就可能重头来过

#### Skills 地址
https://skills.sh/sundial-org/awesome-openclaw-skills/clawdbot-backup

#### 安装命令
```bash
npx skills add sundial-org/awesome-openclaw-skills@clawdbot-backup -g -y
```

#### OpenClaw 手动用法
现在执行一次全量备份，并生成恢复演练清单。

#### 自动触发时机（建议）
- 定时：每天凌晨 02:30 定时触发
- 手动：每次重大改动后再手动补一次

---

## 3) 推荐安装顺序（和视频一致）

### 安装顺序
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

### 分组说明

#### 第一批：基础能力（1-4）
- **skill-vetter**：安全基础，优先安装
- **self-improving-agent**：持续改进机制
- **obsidian-cli**：知识管理基础
- **playwright-scraper**：数据获取能力

#### 第二批：工作流增强（5-7）
- **github-automation**：开发工作流
- **execution-accelerator**：执行效率
- **agentmail**：身份隔离

#### 第三批：协同与集成（8-9）
- **notion + feishu**：团队协同
- **macos-native**：系统集成

#### 第四批：安全保障（10）
- **clawdbot-backup**：容灾备份

### 安装建议
✅ 按顺序逐个安装
✅ 每安装一个就测试基本功能
✅ 确保无问题后再继续下一个
❌ 避免一次性安装所有技能（难以排查问题）

---

## 4) 安装后自检

### 基础检查命令

```bash
# 检查已安装的技能
npx skills check

# 更新已安装的技能
npx skills update
```

### 可选操作

#### 搜索替代技能
```bash
npx skills find <关键词>
```

#### 浏览技能目录
访问 https://skills.sh/ 查看更多技能

---

## 5) 给新手的一句结论

> **先把这 10 个装齐，再去追求复杂玩法。这样做的价值不是"看起来高级"，而是减少返工、提高稳定性、让结果可复用。**

### 核心价值
- 🛡️ **安全第一**：skill-vetter 确保安装安全
- 📈 **持续改进**：self-improving-agent 避免重复踩坑
- 📚 **知识沉淀**：obsidian-cli 建立知识体系
- 🔄 **自动化**：减少机械重复工作
- 👥 **协同效率**：与团队无缝对接
- 💾 **容灾保障**：备份机制防止数据丢失

### 学习路径
1. **安装期**（1周）：按顺序安装 10 个技能
2. **熟悉期**（1月）：每个技能至少使用 3 次
3. **精通期**（3月）：组合使用多个技能解决问题
4. **创造期**（长期）：自定义技能并分享给社区

---

## 📚 相关资源

### 官方资源
- OpenClaw 文档：https://docs.openclaw.ai
- skills.sh 官网：https://skills.sh/
- GitHub 仓库：https://github.com/openclaw/openclaw

### 社区资源
- Discord 社区：https://discord.com/invite/clawd
- ClawHub 技能市场：https://clawhub.com

### 技能链接
所有技能地址：https://skills.sh/ （搜索技能名称）

---

## 📌 标签
#OpenClaw #技能安装 #新手教程 #必装技能 #10大技能

---

**创建时间**：2026-03-09
**最后更新**：2026-03-09
**文档来源**：飞书云文档
**整理者**：小龙虾 🦞
**原作者**：抖音@睡觉的大提琴
