---
tags: [AI, OpenMausBot, 多Agent, Grok平替, 聊天式Agent, Claude, Codex, 视频归档]
---

# OpenMausBot：一个人拥有一整支 AI 团队

> 来源：猴子AI笔记·GitHub案例分享 第57集｜抖音 2026-09-11 19:00
> 视频：https://www.douyin.com/video/7684123584347884851
> GitHub：[milind-soni/OpenMausBot](https://github.com/milind-soni/OpenMausBot)（归档日实测 3584⭐ / 614 forks / TypeScript / Apache-2.0 / 2026-08-11 创建，v0.1.88 2026-09-26 发布）
> 标签：AI, OpenMausBot, 多Agent, Grok平替, 聊天式Agent, 视频归档

---

## 概述

猴子AI笔记第57集介绍 **OpenMausBot**——Grok Bot 的开源平替，上线一个月 2000+ Star（归档日已 3584⭐）。

一句话定位：**把 IM 聊天软件的交互形态搬到 AI Agent 上——每个 bot 是侧边栏里的一位"联系人"，独立人格、独立模型、独立记忆、独立电脑、独立应用权限。** 一个人就能拥有一整支各司其职的 AI 团队。

作者 milind-soni 独立维护，TypeScript（React 19 + Electron + Vite），Apache-2.0，三平台安装包（macOS 签名公证 / Windows / Ubuntu 24.04）。

## 核心特性

### 1. 每个联系人 = 一个真实 Agent
- bot 跑在本机已装好的 `claude` / `codex` / `grok` CLI 上——**用你已有的登录和订阅**，不新开账号、没有中间代理
- 每个 bot 可配：人格（system prompt）、模型（Claude/Codex 并排选、对话中途可切）、记忆（线程级）、电脑、连接的应用
- 对 bot 的操作像管理聊天：右键置顶/标未读/复制会话/复制 bot/删除

### 2. 每个 bot 有一台电脑
- 云 Linux 桌面（Box/boat.dev，面板里实时预览屏幕，可浏览器接管）
- 隔离本地 VM；或平台安全边界认证后**用你这台电脑**（macOS/Ubuntu Xorg 需显式 opt-in，Wayland 保持 fail-closed）
- 500+ 应用通过 Composio Sessions 接入（Gmail/Slack/GitHub/Notion/Linear…，OAuth 一次全员可用）

### 3. 敏感操作必须审批
- Shell 命令、文件编辑、提问都以**内联卡片**出现在聊天里：Allow / Deny / 直接回答
- 权限中介（permission broker）把每个危险动作变成用户决策，云电脑和本地电脑一视同仁
- 密钥只写不读：UI 只能看到"已配置"标志

### 4. 频道 + 团队导入
- **频道隔离**：Work / Personal / 每个项目独立频道，各有自己的 transcript、共享指令、工作目录、bot 名册
- **Markdown 一键装团队**：从 [BotMRR](https://botmrr.io) 浏览成品团队或本地 .md / GitHub URL 导入——读 YAML frontmatter 创建 bots + Chief of Staff + 频道 + playbook；纯 Markdown 部分 Grok/Claude/ChatGPT/人都能读（格式可移植）
- 安全默认：连接默认关、routine 默认暂停、包里永不含密钥/会话/权限/电脑访问

### 5. 语音与表达
- 每个 bot 可配独立语音（ElevenLabs / Fish Audio / Grok xAI / Mac 内置 / 本地 Chatterbox），早餐时听昨晚跑完的活
- **Call 模式**：语音对话，边干边说，口头请求审批

## 架构（双进程）

```
App(React+Tailwind, 5199) ──HTTP命令──▶ Harness Server(127.0.0.1:8799)
        ◀────一条 SSE 事件流────────────────  │
                                              ├─ Driver Registry: claude/codex/grok CLI + 云电脑
                                              ├─ Permission Broker（审批卡）
                                              ├─ Box API(云电脑) / Composio(500+应用)
                                              └─ TTS(ElevenLabs/Fish/xAI/Mac/Chatterbox)
```

- **App 不持有任何传输层**：只发类型化命令、fold SSE 流入 state——服务端 backed store，一个 reducer，客户端零传输
- **Driver 层归一化**：每 provider 一个 driver（stream-JSON / JSON-RPC / ACP），把各家原生协议归一成一条 canonical 事件流（NDJSON 落盘）；未知 driver 降级为"不可用"，不炸舰队
- **Local-first**：transcript、密钥、事件全在 `~/.openmausbot`，不上云
- **MCP 控制面**：内置 stdio MCP server，让 Claude Desktop/Cursor 能编排 OpenMausBot——可查 bot/读 transcript/派活/等完成/切模型/打断，但**刻意不暴露**审批授权、删除、凭据、电脑生命周期

## 战略分析

### 与知识库现有体系的对照

| 维度 | OpenMausBot | 知识库相关归档 | 说明 |
|---|---|---|---|
| 形态 | IM 式多 bot 聊天 | [[技能文档/2026-04-20-Multica-AIAgent项目管理平台]] 项目管理式 | 一个管"聊"，一个管"项目" |
| 平替目标 | Grok Bot 开源版 | [[技能文档/2026-03-17-一天94次提交！我用OpenClaw指挥Claude Code，3天搭出一套AI开发团队]] | OpenClaw 生态同样做"AI 团队"叙事 |
| 隔离 | 独立电脑/VM/审批门 | [[技能文档/2026-09-04-OpenBot-每Agent独立电脑安全网关]] | OpenBot 也做"每 Agent 一台电脑"，对比看 |
| Harness | 双进程 + Driver 归一 | [[技能文档/2026-08-27-Prime-Agent-自进化Harness框架]] | Prime 重自进化，OpenMausBot 重交互形态 |

### 值得借鉴的点

1. **"聊天软件"是 AI 团队的正确交互隐喻**——把 Agent 塞进万能助手框被证明是错的形状；把每个 Agent 变成"联系人"后，频道隔离、置顶、@、审批卡全部自然长出来。这个交互范式对 InStreet 等社交型 Agent 平台有直接参考价值
2. **Driver 归一化层**——不同 Agent CLI（stream-JSON/JSON-RPC/ACP）归一成一条事件流 + 未知 driver 优雅降级。任何想聚合多家 Agent 的项目都需要这层，和 Hermes 的多 provider 思路同源
3. **审批卡 + 密钥只写不读 + 连接默认关**——权限产品设计的三件套范本，比"沙箱一刀切"体验好
4. **团队即 Markdown（可移植格式）**——YAML frontmatter 机器读、正文人/Agent 都能读，BotMRR 还做了分发市场。团队模板 = 新资产类别
5. **MCP 控制面"刻意有界"**——能派活但绝不能替你批准。安全边界划得克制，值得抄

### 风险与边界

- 单作者（milind-soni）维护，3584⭐ 但 614 forks 里已有 Murage 等分叉，生态分裂风险
- README 顶部反加密货币声明——出圈后已被币圈蹭名，社区治理压力
- 评论区灵魂拷问未解：与 OpenClaw/Hermes/龙虾/Multica 的差异化边界仍模糊，"同类产品太多"
- 本机 CLI 依赖：claude/codex/grok 订阅是硬门槛

## 归档日志

- 2026-09-27 归档（抖音视频页免登录提取元数据+章节要点；GitHub API 实测 3584⭐/Apache-2.0/v0.1.88；README 26K 全文精读）
