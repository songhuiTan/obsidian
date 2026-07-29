---
source: 微信公众号
author: AI工具日报
account: 小奇的干货日记
date: 2026-07-05 15:32
url: https://mp.weixin.qq.com/s/JjtZLLICQ6-um6ASH_Pblg
tags: Emdash, Task Master, mcpm, mception, MCP安全, Agent并行, 任务拆解, AI编码, Git worktree
---

# AI实用工具：Emdash/Task Master/mcpm — AI编码工作流三件套

> 三个工具分别卡在 AI 编码工作流中三个被忽视的环节：**写完代码之前的任务拆解、写完代码之后的 Agent 并行编排、以及所有 Agent 都依赖的 MCP 服务器的安全审计。**

---

## 一、Emdash — 31 个 Agent 并行跑，互不打架

> YC W26·Apache 2.0·⭐ 5,042·60K+ 下载·110 贡献者·130 releases

**问题：** 一次只能跑一个 Agent，多个 Agent 共用工作目录，同时编辑同一文件立刻冲突。

**解法：Git worktree 隔离。** 每个任务从基础分支创建一个独立 worktree，Agent 互不干扰。内置 diff 视图逐文件对比改动，满意合并，不满意丢弃。

**底层工程亮点：**
- `WorktreePoolService`：后台预创建 reserve worktree，点「New Task」瞬间就位（免 3-7 秒 `git worktree add`）
- `TaskLifecycleService`：PTY 进程管理 Agent 生命周期，操作去重、优雅退出
- 状态存本地 SQLite（Drizzle ORM），不经过云端

**核心特性：**
- 支持 **31 个 CLI Agent**（Claude Code、Codex、Gemini、Cursor、Qwen Code、OpenCode、Amp、Devin、GitHub Copilot 等）
- 各 Agent 用各自 API Key，Emdash 不抽成
- **Issue Tracker 集成**：Linear/Jira/GitHub Issues/GitLab/Asana 等 9 种，工单直丢 Agent
- **远程 SSH/SFTP**：笔记本操作界面，Agent 跑在云端编译

**安装：**
```bash
brew install --cask emdash        # macOS
# Windows/Linux: GitHub Releases
```

**对比：** Anthropic Dynamic Workflows 只在 Claude 内部做并行，模型限 Claude。Emdash 是 provider-agnostic 调度层，可混用不同厂商 Agent。

---

## 二、Task Master — 把 PRD 拆成带依赖图的任务树

> ⭐ 27K+·60 贡献者·92 releases·npm

**问题：** Agent 上下文窗口有限，需求越长越容易在中间丢失关键上下文。

**解法：** 给 PRD 文件 → LLM 拆成带依赖关系的结构化任务树（JSON 文件），每个任务有 ID/标题/描述/验收标准/依赖列表。Agent 每次只看当前任务，不需要把整个 PRD 塞进上下文。

**36 个 MCP 工具分三档按需加载：**
| 模式 | 工具数 | 适用场景 |
|------|--------|---------|
| Core | 7 | 最基本的任务 CRUD，省 70% token |
| Standard | 15 | 加项目初始化和复杂度分析 |
| Full | 36 | 研究、标签、依赖图等全部能力 |

**多模型角色分工：** 配三个模型角色——主模型（任务拆解/代码生成，如 Claude Opus）、研究模型（Web 搜索，如 DeepSeek）、fallback 模型。不同任务自动路由到不同模型。

**Autopilot 模式：** `tm autopilot` 启动自主循环：生成测试→实现代码→跑测试→提交→取下一个任务。七个专门 MCP 工具驱动。

**安装：**
```bash
npm install -g task-master-ai
claude mcp add taskmaster-ai -- npx -y task-master-ai
task-master parse-prd your-prd.txt   # 解析 PRD 生成任务
task-master next                      # 查看下一个待办
task-master expand-task --id=3        # 拆子任务
```

---

## 三、mcpm + mception — MCP 服务器安全审计

**问题：** MCP 生态正在经历 npm 早期阶段——人人都在装，没人审查。npm 等了四年才来 npm audit（2018 年），MCP 不该再等。

### mcpm — MCP 包管理器 + 运行时护栏

> MIT·npm `@getmcpm/cli`·v0.14.0

- **🔍 搜索** — `mcpm search filesystem`
- **🔐 信任评分** — `mcpm info` 查看 0-100 四维评分和详情
- **🛡️ 运行时防护** — `mcpm guard` 作为 stdio 中继层，实时扫描工具描述/响应/参数中的 OWASP MCP Top 10 攻击模式。安装时快照 schema，运行时检测 schema drift 立即阻断
- **👥 团队共享** — `mcpm export` → `mcpm lock` → `mcpm up` 一键复现 MCP 环境，stack file 带信任策略 (`minTrustScore: 60`)

```bash
npm install -g @getmcpm/cli
mcpm search filesystem
mcpm info io.github.domdomegg/filesystem-mcp
mcpm install io.github.domdomegg/filesystem-mcp
mcpm audit          # 审计所有已装服务器
mcpm doctor         # 诊断配置健康状态
mcpm export > mcpm.yaml && mcpm lock && mcpm up
```

### mception — MCP 安全审计引擎

- **深度静态审计**：SAST + SCA + 依赖溯源
- 覆盖 Python/TypeScript/JavaScript/Go/Rust/Ruby 六种语言
- **60+ 检测规则**，按严重度分类
- 适合 **CI 门禁**场景

**两者互补：** mception 管「装之前深度检查」，mcpm 管「装之后持续防护」。CI 用 mception，日常工作用 mcpm。
