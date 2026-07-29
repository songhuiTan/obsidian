# Pi Package 生态入门——从安装到社区精选 | Pi SDK 系列第 14 篇

> **来源**：mijack / 麦坊（微信公众号）
> **日期**：2026-07-06
> **系列**：5 周从入门到精通 Pi SDK 系列第 14 篇
> **原文链接**：https://mp.weixin.qq.com/s/HnoPFD-HWKG5MKcvoLR0TQ

## 核心要点

Pi Package 是 Pi Agent 的扩展分发机制。Pi 默认只装四个工具（read/write/bash/edit），其他能力全部通过 Package 注入。社区已发布 **4300+** 个 Package。

## Package 类型

| 类型 | 说明 | 开发成本 |
|------|------|---------|
| **Extension** | TypeScript 模块，注册工具/命令/事件钩子 | 最高（需写 TS） |
| **Skill** | 渐进式加载的能力单元（SKILL.md） | 中 |
| **Prompt Template** | .md 提示词模板 | 低 |
| **Theme** | 终端主题配置 | 最低 |

## 安装与管理

```bash
# 三种来源
pi install npm:pi-web-access          # npm（最常用）
pi install github:user/repo           # GitHub（无 // 前缀）
pi install ./my-extension             # 本地路径（开发调试）

# 管理命令
pi list
pi update                             # 更新 Pi 自身
pi update --all                       # 更新所有
pi update npm:pi-web-access           # 精确更新
pi remove npm:pi-web-access           # 卸载

# 安全控制
pi --no-extensions                    # 完全禁用 Extension
pi --exclude-extensions <name>        # 排除特定 Extension
```

> ⚠️ **安全**：Package 拥有与 Pi 进程相同的系统访问权限。安装前务必审查源码。

## 多 Package 冲突处理

**核心原则：加载顺序决定优先级**。Pi 目前没有命名空间隔离和冲突检测警告。

- **覆盖式资源**（后注册覆盖先注册）：Tool、Command、Prompt Template、Skill、Theme
- **多播式资源**（全部执行）：Event 钩子

**最佳实践**：避免装功能重叠的 Package，用 `--exclude-extensions` 精确控制，注意加载顺序。

## 社区精选 Package（4300+）

### 🔍 搜索与信息获取
| Package | 下载量 | 说明 |
|---------|--------|------|
| **pi-web-access** | 110K/mo | 网页搜索、URL 抓取、GitHub 克隆、PDF 提取、YouTube 理解 |
| **@juicesharp/rpiv-web-tools** | 13.5K/mo | 可插拔搜索，支持 10+ 搜索引擎 |
| **@ollama/pi-web-search** | 16K/mo | Ollama 内置 Web 搜索 API |

### 🧠 记忆与上下文管理
| Package | 下载量 | 说明 |
|---------|--------|------|
| **pi-hermes-memory** | 12.2K/mo | 持久记忆 + 会话搜索 + 密钥扫描 |
| **context-mode** | 94.2K/mo | 节省 98% 上下文窗口！沙箱 + FTS5 知识库 |
| **@hypabolic/pi-hypa** | **203K/mo** | 下载量第一！自动压缩 Shell 输出，省上下文 |

### 🤖 子 Agent 与工作流
| Package | 下载量 | 说明 |
|---------|--------|------|
| **pi-subagents** | 86.5K/mo | 委托子 Agent，链式/并行执行 |
| **@quintinshaw/pi-dynamic-workflows** | 18K/mo | Claude-Code 风格动态工作流，100+ 子 Agent 并行 |
| **pi-crew** | 14.1K/mo | 协调 AI 团队、worktree、异步编排 |

### 🛡️ 安全与代码质量
| Package | 下载量 | 说明 |
|---------|--------|------|
| **@vigolium/piolium** | 32K/mo | 多阶段安全审计 |
| **pi-simplify** | 21.2K/mo | 审查代码清晰度、一致性、可维护性 |
| **pi-lens** | 26.2K/mo | 实时代码反馈（LSP/Linter/Formatter/类型检查） |

### 🧰 开发效率工具
| Package | 下载量 | 说明 |
|---------|--------|------|
| **@juicesharp/rpiv-todo** | 47.8K/mo | Agent Todo List，实时渲染叠加层 |
| **@juicesharp/rpiv-ask-user-question** | 56.7K/mo | 结构化问卷，Agent 需猜测时向用户提问 |
| **@ayulab/pi-rewind** | 28.9K/mo | /rewind 检查点导航，随时回到之前状态 |

## 日常效率环境推荐组合

```bash
# 基础
pi install npm:pi-web-access
pi install npm:@hypabolic/pi-hypa       # 或 context-mode
pi install npm:pi-subagents

# 效率工具
pi install npm:@juicesharp/rpiv-todo
pi install npm:@juicesharp/rpiv-ask-user-question
pi install npm:pi-lens
pi install npm:pi-simplify
pi install npm:@ayulab/pi-rewind
```

## 开发 Package

目录结构：
```
my-pi-extension/
├── package.json       # npm 包定义 + pi 配置
├── tsconfig.json
├── src/index.ts       # Extension 入口
├── prompts/*.md       # Prompt Template（可选）
└── README.md
```

`package.json` 关键字段：`keywords` 必须包含 `"pi-package"` 才能在 Catalog 中收录。<br>
发布：`npm publish --access public`
