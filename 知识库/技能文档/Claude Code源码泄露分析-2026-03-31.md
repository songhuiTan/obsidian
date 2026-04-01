# Claude Code源码遭泄露分析

> 来源：https://mp.weixin.qq.com/s/ssxPswUL-eVdhkcBuwPP_w
> 作者：鲁工（AI编程实验室）
> 归档：2026-03-31

## 泄露原因

Anthropic发布Claude Code的npm包时，把source map文件一起打包进去了。Source map是JS调试文件，把压缩混淆后的代码映射回原始源码。本应在`.npmignore`或`package.json`的`files`字段里排除，结果都没有。

- 一个60MB的`cli.js.map`文件暴露在npm包中
- **1906个文件，512000多行代码**全部暴露
- GitHub存档仓库：https://github.com/instructkr/claude-code（5400+ Star）

**注意：这不是第一次**。2025年2月Claude Code刚发布时就因同样原因泄露过，Anthropic删了旧包去掉了source map，结果现在又来一遍。

## 技术栈

| 组件 | 技术 |
|------|------|
| 语言 | TypeScript |
| 运行时 | Bun |
| 终端UI | React + Ink |
| CLI解析 | Commander.js |
| Schema校验 | Zod v4 |

## 核心架构发现

### 工具系统
- 40多个工具，每个独立模块
- 有自己的输入Schema、权限模型和执行逻辑
- 关键工具：BashTool（Shell命令）、FileReadTool（读文件）、GrepTool（内容搜索，底层ripgrep）、AgentTool（派生子Agent）

### 命令系统
- 50多个斜杠命令
- 覆盖/commit、/review、/vim、/doctor等开发工作流

### Feature Flag（最有意思的部分）
通过Bun编译时常量折叠，未开启功能被完全消除：

| Flag | 疑似功能 |
|------|----------|
| KAIROS | 助手模式 |
| PROACTIVE | 主动模式 |
| BUDDY | ASCII电子宠物（愚人节彩蛋） |
| TORCH/TUNGSTEN/FENNEC | 内部测试功能 |

内部员工用`process.env.USER_TYPE === 'ant'`区分，ant用户可用ConfigTool、TungstenTool、REPLTool等额外工具。

## 安全审查发现

### Bug 1：Plan文件白名单匹配过宽
用`startsWith`做前缀匹配，如planSlug是`blue-fox`，则`blue-fox-backup.md`、`blue-fox-evil.md`都会被当成合法Plan文件，绕过权限检查。

### Bug 2：写文件只处理一层symlink
代码想"写穿symlink同时保留链接本身"，但只`readlinkSync`一次。多级链接链会导致中间symlink被普通文件替换。

### Bug 3：WebSocket重连回放不一致
Node和Bun运行时下行为不一致，可能导致断线重连后消息丢失。

> 权限系统本身经过高强度安全补丁演进，对UNC路径、Shell展开、symlink逃逸、glob绕过都有防御。问题出在系统复杂度本身。

## 值得学习的工程优化

**main.tsx启动优化：**
- 前20行在任何import前触发MDM配置读取和macOS钥匙串预取
- 利用模块加载的~135ms窗口做并行预热，节省65ms启动时间
- 重型模块（OpenTelemetry ~400KB，gRPC ~700KB）全部动态import延迟加载

## 结论

- 泄露的是CLI客户端代码，**不涉及模型权重和用户数据**
- 对普通用户无直接影响
- 同一问题犯两次，CI/CD流水线加个check就能防住
- 教训：发npm包前永远检查`.npmignore`和`files`字段

## 标签

#claude-code #源码泄露 #安全 #工程化 #npm
