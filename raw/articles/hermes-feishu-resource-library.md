---
source_url: https://my.feishu.cn/wiki/DbDhwhYcFiHY8JkpX0fcAYdMnve
ingested: 2026-05-05
sha256: 960dd7c77538f34369aad1502832efe2a6678aeffa38d884c55ead4b3d9c8844
title: Hermes Agent 详细教程（12章全）
---

---
title: Hermes Agent 详细教程（12章全）
source: https://my.feishu.cn/wiki/DbDhwhYcFiHY8JkpX0fcAYdMnve
archived: 2026-05-05
tags: [Hermes, Agent, 教程, 飞书归档]
description: 飞书文档归档 - Hermes Agent 完整教程，涵盖安装部署、核心功能、技能开发、多平台接入、MCP扩展等12章
---

# Hermes Agent 详细教程（12章全）

> 来源：飞书文档归档 | 原始链接：https://my.feishu.cn/wiki/DbDhwhYcFiHY8JkpX0fcAYdMnve
> 归档时间：2026-05-05


## 
Hermes Agent 是 Nous Research 在 2026 年推出的开源自改进 AI Agent（MIT 协议），核心亮点是内置闭环学习系统：它能从经验中自动创建技能（Skills）、持续优化、跨会话持久记忆，并构建对用户的深度模型。它不是简单的聊天机器人或 IDE 插件，而是一个真正能“越用越聪明”的持久化自治代理，支持 CLI、Telegram、Discord 等 15+ 平台，兼容任意模型（OpenRouter、Nous Portal、Ollama、本地 vLLM 等），内置 40+ 工具，还支持 MCP、语音、子代理、定时任务等。

[多维表格 - 请到飞书网页端查看]
通过网盘分享的文件：
链接: https://pan.baidu.com/s/1B0ureVdeW8ave8zdtxCC-g?pwd=98iy 提取码: 98iy 复制这段内容后打开百度网盘手机App，操作更方便哦 
--来自百度网盘超级会员v3的分享
我用夸克网盘给你分享了「hermesagent（记得保存持续更新）」，点击链接或复制整段内容，打开「夸克APP」即可获取。手册以及视频教学都放网盘里面了
/~48643YH9Ev~:/
链接：https://pan.quark.cn/s/904c76c78a22

## 

### 
在 AI Agent 领域，2026 年上半年最引人注目的新星莫过于 Hermes Agent（由 Nous Research 推出，MIT 开源协议）。它不是又一个“会聊天的模型包装器”，而是一个真正能持续进化的自治系统。
- 传统聊天机器人（如 ChatGPT、Claude、Gemini）：每次对话都是独立的，记忆仅限于当前会话窗口。它们擅长单次任务，但“健忘”——你昨天告诉它的项目细节，今天可能完全不记得。它没有真正的“成长”机制。
- OpenClaw（此前最流行的开源本地 Agent）：强调“控制平面 + 人工编写技能”的架构，功能强大，但技能大多需要开发者手动维护，缺乏自动从经验中提炼和优化的闭环。
- AutoGen / CrewAI / LangGraph 等框架：擅长多代理协作和复杂工作流编排，但通常是无状态或短期记忆的，需要开发者自己设计记忆存储、技能复用和长期学习机制，部署和运维成本较高。
- Hermes Agent 的核心差异：它内置了闭环学习系统（Closed Learning Loop）。每完成一个任务，它都会自动分析成功/失败轨迹，自主生成可复用的技能（Skills），将经验固化成 Markdown 文件，下次遇到类似问题时直接调用并持续优化。同时，它拥有多层持久记忆（跨会话、全文搜索 + LLM 总结 + 用户画像建模），真正实现了“越用越聪明”。
一句话总结：其他 Agent 是“工具”，Hermes Agent 是“会成长的数字员工”。

### 
Hermes Agent 的独特价值集中体现在以下三个相互强化的机制上：
1. 技能自动生成与自改进
   它不是靠人工写 Prompt 或技能，而是通过“成功轨迹提炼”机制：当它完成一个复杂任务（如“帮我分析 GitHub 仓库并生成部署脚本”）后，会自动将整个思考过程、工具调用序列和最终方案提炼成结构化技能文件（存放在 `~/.hermes/skills/`）。下次你说类似需求，它会优先复用并迭代这个技能。官方宣称这是“目前唯一内置完整学习闭环的 Agent”。
2. 多层持久记忆系统
   - 短期：当前会话上下文
   - 中期：SQLite 全文搜索 + 向量检索
   - 长期：Honcho 用户建模（跨会话构建对你的偏好、项目风格、工作习惯的深度画像）
   你可以随时让它“搜索我上周关于 Kubernetes 的所有讨论”，它能精准召回并总结。
3. 用户模型与个性化
   通过持续交互，Hermes 会越来越懂你——你的写作风格、代码偏好、技术栈、甚至沟通语气。它支持通过 `SOUL.md` 全局人格文件和项目级上下文文件进行深度定制。
此外，它还具备以下硬核能力：
- 模型完全无关：支持 OpenRouter（200+ 模型）、Nous Portal、Ollama、本地 vLLM、Anthropic、OpenAI、NVIDIA NIM 等任意 OpenAI 兼容端点，一条命令即可切换（`hermes model`）。
- 40+ 内置工具：网页搜索、浏览器自动化、视觉理解、图像生成、TTS、终端执行、文件操作、代码解释器等，开箱即用。
- 多平台统一接入：CLI、Telegram、Discord、Slack、WhatsApp、Signal、Email 等 15+ 平台通过单一网关实现，手机上就能随时指挥它在云服务器上干活。
- 灵活部署：支持本地、Docker、SSH、$5 VPS、甚至 Modal/Daytona 等 serverless（闲置几乎零成本）。

### 
Hermes Agent 特别适合以下几类用户：
- 个人生产力伴侣：每天早上自动给你发“昨日项目进度 + 今日待办 + 相关论文摘要”，并在你睡觉时帮你处理备份、邮件分类。
- 研究与内容工作者：让它持续跟踪某个技术领域，自动抓取最新论文、生成阅读笔记、甚至帮你写技术博客草稿。
- 开发者与 DevOps：作为 24/7 在线的代码审查员、部署助手、Bug 调试伙伴。它能记住你整个项目的架构和历史决策。
- 团队共享 Bot：部署在公司 Slack/Telegram 群里，作为知识库 + 自动化执行者（会议纪要生成、数据报表推送）。
- 长期项目管理者：跨月、跨年的项目伴侣——它记得你半年前定下的技术选型理由，并会在你偏离时温和提醒。
一句话：只要你需要一个“不会忘事、会自己变聪明、能 24 小时在线工作”的数字员工，Hermes Agent 就是目前最接近这个目标的开源方案。

### 

### 
Hermes Agent 自 2026 年 2-3 月正式发布以来，迅速在开发者社区掀起热潮：
- GitHub 仓库（NousResearch/hermes-agent）星标数已超过 6 万（并持续快速增长）。
- 官方 Discord 社区活跃度极高，每天都有新技能分享和部署案例。
- 已有大量用户从 OpenClaw 迁移，理由正是“终于有了真正的自学习能力”。
- 生态正在快速完善：agentskills.io 技能分享平台、MCP 服务器生态、Voice Mode、ACP 编辑器集成等功能陆续落地。
未来展望：随着更多用户贡献技能和 MCP 工具，Hermes 有望成为开源 Agent 领域的“事实标准”。Nous Research 本身在 Hermes 模型家族和 Atropos RL 框架上的积累，也让它在长期智能体训练方向上具备独特优势。
---
本章小结：  
Hermes Agent 不是“另一个 AI 工具”，而是第一个真正把“持续学习与成长”做成核心架构的开源自治代理。如果你厌倦了每次都要重新给 AI“喂上下文”、厌倦了技能需要手动维护、厌倦了 Agent 像一次性工具一样用完就忘，那么 Hermes Agent 值得你花时间深入学习。
下一章我们将手把手带你完成60 秒安装 + 首次配置，让你在 5 分钟内拥有一个真正属于自己的“会成长的数字员工”。

## 
欢迎来到 Hermes Agent 的实战部分！本章的目标非常明确：让你在 5 分钟内完成安装，并运行起第一个可靠的对话。我们严格遵循官方推荐流程，同时补充大量实操细节和避坑指南。

### 
官方支持的操作系统：
硬件与模型前置要求（非常重要！）：
强烈建议：先准备好一个支持 64K+ 的 API Key（OpenRouter、Nous Portal、Anthropic 或 OpenAI 均可）。没有 Key 也可以先用 Ollama 本地模型测试。

### 
这是 Hermes Agent 最被称道的“零门槛”安装方式。

#### 
打开终端，复制粘贴以下命令并回车：
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
安装脚本会自动完成：
安装完成后，务必重新加载 shell 配置：
source ~/.bashrc     # 如果你用 bash
# 或
source ~/.zshrc      # 如果你用 zsh

#### 
hermes --version
正常输出示例：
Hermes Agent v0.11.0 (commit: xxxxx)
如果提示 command not found，请重新执行 source 命令，或重启终端。

### 

#### 

#### 
安装脚本会自动检测架构并使用对应二进制，无需额外操作。

#### 
Termux 安装流程略有不同，建议参考官方文档中的 Termux 专属指南。功能基本可用，但部分工具（如浏览器自动化）受限。

#### 

### 
安装完成后，不要急着连接 Telegram，先确保基础对话能跑通。

#### 
hermes setup
这个命令会引导你完成：

#### 
hermes model
这是一个交互式菜单，按提示选择即可。推荐新手路径：
配置会分别保存在：

#### 
推荐使用现代 TUI 界面（体验更好）：
hermes --tui
或者经典 CLI：
hermes
测试问题示例（建议先用这个验证）：
请用 5 个 bullet point 总结一下当前目录下有哪些文件，并告诉我最大的文件是什么。
如果能正常回复并调用工具，说明安装成功！

#### 
遇到任何问题，先运行：
hermes doctor
它会自动检查：

### 

#### 
hermes update
或重新执行安装脚本（脚本会自动检测并升级）。

#### 
rm -rf ~/.hermes
# 同时删除 shell 补全（按需）

### 
重要提醒：  只有当你能稳定完成一次完整对话后，才建议继续设置 Telegram/Discord 网关、安装技能或开启语音模式。很多用户跳过这一步导致后续问题频发。
本章小结  恭喜！你现在已经拥有了一个可以本地运行的 Hermes Agent 实例。整个过程真正做到了“60 秒安装 + 5 分钟可用”。
下一章预告：  第 3 章：首次配置与模型选择（最关键一步）  我们将深入讲解如何选择最适合你的模型、优化配置参数、设置成本控制策略，并分享新手最容易踩的几个坑。

## 

在前两章中，我们已经成功安装了 Hermes Agent 并运行了第一次对话。现在是时候进行最重要也最容易被忽视的步骤——模型与提供商的精细配置。选择合适的模型和提供商，直接决定了 Hermes 的智能上限、响应速度和长期使用成本。
本章将手把手带你完成从“能用”到“用得爽”的配置过程。

### 
这是 Hermes 最人性化的配置入口。无论你是新手还是老鸟，都推荐通过这个命令完成首次配置。
运行命令：
hermes model
你会看到一个清晰的交互式菜单，大致流程如下：
小技巧：如果你之前已经配置过，可以随时重新运行 hermes model 来切换模型，整个过程无需重启。

### 
以下是根据不同需求给出的推荐路径（2026 年 4 月最新情况）：
新手建议：先用 OpenRouter 或 Nous Portal，因为它们支持一键切换模型，且有完善的降级策略。

### 
Hermes 把配置清晰地分为两类文件：

#### 
# 查看内容（请妥善保管）
cat ~/.hermes/.env
典型内容：
OPENROUTER_API_KEY=sk-or-xxxxxxxxxxxxxxxxxxxx
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxx
NOUS_PORTAL_API_KEY=xxxxx

#### 
这是核心配置文件，推荐用编辑器打开查看：
nano ~/.hermes/config.yaml
# 或
vim ~/.hermes/config.yaml
关键字段说明：
model: openrouter/anthropic/claude-4-opus          # 当前默认模型
terminal:
  backend: docker                                  # 推荐 docker（安全性高）
  timeout: 300
memory:
  enabled: true
  max_tokens: 32000
skills:
  auto_create: true                                # 允许自动生成技能
  storage_path: ~/.hermes/skills
voice:
  enabled: false
gateway:
  enabled: false                                   # 暂时关闭，等第6章再开
通过 CLI 快速修改配置（推荐方式）：
# 设置模型
hermes config set model anthropic/claude-4-opus

# 设置终端后端为 Docker
hermes config set terminal.backend docker

# 设置最大上下文
hermes config set memory.max_tokens 48000

### 
以下参数直接影响使用体验，建议新手按以下顺序调整：
修改示例：
hermes config set temperature 0.4
hermes config set max_tokens 12000

### 
Hermes 最强大的地方在于随时无缝切换模型，无需重启。

#### 
hermes model          # 再次运行向导
# 或直接命令行切换
hermes config set model openrouter/google/gemini-2.5-pro

#### 
避坑提醒：

### 
本章小结  配置是 Hermes 使用体验的分水岭。选对模型和参数后，你会明显感觉到 Hermes 的“聪明程度”大幅提升。

## 

在前三章中，我们已经完成了安装、首次对话和模型配置。现在是时候深入 Hermes Agent 的灵魂——它到底为什么能“越用越聪明”？
本章将系统讲解 Hermes 的三大核心机制：记忆系统、技能系统和工具系统，并穿插真实案例和可立即执行的命令，让你不仅“知道它是什么”，更能“感受到它在工作”。

### 
传统 AI 的最大痛点就是健忘。Hermes 彻底解决了这个问题，它拥有三层记忆架构：

#### 

#### 

#### 
实战案例：
你连续三天让 Hermes 帮你分析不同项目的代码架构。第四天你说：“帮我设计一个统一的微服务模板”，它会自动回忆前几天的讨论，结合你的偏好（喜欢用 FastAPI + PostgreSQL + Docker）给出高度定制化的方案。
关键命令：
hermes memory list          # 查看所有持久化记忆条目
hermes memory search "关键词"
hermes memory clear --old   # 清理旧记忆（谨慎使用）

### 
这是 Hermes 与其他 Agent 最大的区别：它会自己写技能、自己进化。

#### 
技能是一个结构化的 Markdown 文件，包含：

#### 

#### 
# 搜索社区技能
hermes skills search kubernetes

# 安装一个技能
hermes skills install openai/skills/k8s-deployment

# 查看已安装技能
hermes skills list

# 在对话中手动触发技能学习（高级）
/learn
真实案例：
第一次让 Hermes “帮我部署一个带监控的 FastAPI 项目到 VPS”。它花了 12 分钟完成（中间试错 3 次）。成功后，它自动生成了 fastapi-production-deploy.md 技能文件。
一个月后你说同样需求，它30 秒内就完成了，而且比上次更规范（多了健康检查和自动备份）。
技能文件位置：
ls ~/.hermes/skills/

### 
Hermes 内置 47+ 工具，分为以下几大类：

#### 
hermes tools list

#### 
hermes tools
这个命令会让你为不同平台（CLI、Telegram、Discord）分别开启/关闭工具，避免权限过大。
实战示例（在对话中）：
用浏览器打开 https://github.com/NousResearch/hermes-agent，总结最新 5 个 commit 的改动。
Hermes 会自动调用 browser_automation 工具完成任务。

### 
Hermes 支持递归 spawning 子代理，每个子代理都是一个完整独立的 Hermes 实例，拥有自己的：
使用方式（在对话中）：
请创建一个子代理，专门负责调研最新 AI Agent 框架，并把结果汇总给我。
子代理完成后会把结果返回主代理，形成高效的“团队协作”模式。
这让 Hermes 能同时处理多个复杂任务，而不会互相干扰。

### 
Hermes 非常重视安全，提供了五种终端后端：
安全增强建议：
# 开启命令审批（每次执行危险命令前确认）
hermes config set security.require_approval true

# 限制文件访问范围
hermes config set security.allowed_paths "~/projects,~/Downloads"

### 
本章小结  理解了记忆、技能、工具这三大核心机制后，你就真正掌握了 Hermes 的“思考方式”。这也是它能长期陪伴你、不断进化的根本原因。

## 

在前四章中，我们已经完成了安装、配置和核心概念的学习。现在是时候把 Hermes 的日常操作界面彻底掌握了。Hermes 提供了两种交互方式：经典 CLI 和现代 TUI。熟练使用它们，能让你像使用 Vim 或 Emacs 一样高效地与 Hermes 协作。

### 

#### 
新手建议：直接使用 hermes --tui，体验更好。

#### 
TUI 模式快捷键（推荐记忆）：
经典 CLI 模式快捷键：
启动建议：
# 推荐日常使用
hermes --tui

# 想快速测试或脚本调用
hermes

### 
在对话界面中输入 / 即可触发命令补全。以下是目前最常用的 Slash 命令：
实战技巧：

### 

#### 
Hermes 支持极长的上下文（64K+），你可以放心进行多轮深度讨论：
推荐对话模式：
用户：帮我设计一个完整的用户认证系统（FastAPI + JWT + PostgreSQL）
Hermes：（给出方案）
用户：把数据库部分改成使用 SQLAlchemy 2.0 风格
Hermes：（自动迭代上一个方案）
用户：再加上 Redis 缓存层和限流中间件

#### 

#### 
# 查看所有历史会话
hermes sessions list

# 切换到指定会话
hermes sessions switch <session_id>

# 删除会话
hermes sessions delete <session_id>

# 导出当前会话为 Markdown
hermes sessions export --format md
最佳实践：

### 

#### 
在 TUI 中：

#### 
直接把文件从文件管理器拖进 TUI 窗口，Hermes 会自动识别并分析：

#### 
当 Hermes 调用工具时，TUI 会实时显示：
这让你能清楚看到 Hermes 在“思考什么、做了什么”。

#### 
# 启动时直接加载特定会话
hermes --continue --session project-x

# 以只读模式启动（适合演示）
hermes --readonly

# 开启详细日志（调试用）
hermes --verbose

### 
本章小结  掌握 CLI/TUI 后，你已经可以像专业开发者一样高效地与 Hermes 协作了。接下来，我们将把 Hermes 从“终端里的工具”变成“真正 24/7 在线的数字员工”。

## 

到目前为止，Hermes 还只是一个“坐在你电脑里的聪明助手”。从本章开始，它将变成一个真正的 24/7 在线数字员工——无论你在手机上、平板上，还是在开会时，都能随时通过熟悉的聊天软件指挥它在云服务器上工作。
这是 Hermes 最具革命性的功能之一：单一网关统一接入 15+ 平台。

### 
重要提醒（再次强调）：
只有当你能稳定完成基础对话后，才建议开启网关。很多用户跳过这一步导致后续问题。
启动配置向导：
hermes gateway setup
这个命令会引导你完成以下步骤：
配置完成后，Hermes 会自动在后台运行一个网关服务，所有平台的消息都会统一转发给同一个 Agent 实例。
查看网关状态：
hermes gateway status
停止/重启网关：
hermes gateway stop
hermes gateway start

### 
Telegram 是目前使用 Hermes 最舒服的平台，理由：

#### 
进阶配置示例（手动编辑 ~/.hermes/config.yaml）：
gateway:
  telegram:
    enabled: true
    token: "123456789:AAF..."
    allowed_users:
      - 987654321          # 你的 User ID
    allowed_groups:
      - -1001234567890     # 群组 ID（负数）
    parse_mode: "MarkdownV2"

### 
Discord 示例（最常用第二平台）：
hermes gateway setup
# 选择 Discord → 输入 Bot Token → 输入 Guild ID → 设置允许的频道
多平台同时接入：
你可以在一次 hermes gateway setup 中选择多个平台，Hermes 会自动为每个平台生成对应配置。所有平台共享同一个 Agent 实例和记忆！

### 
Hermes 的网关设计非常优雅：
查看所有已连接平台：
hermes gateway platforms
为不同平台设置不同权限（高级）：
gateway:
  telegram:
    allowed_tools: ["web_search", "terminal", "file_read"]
  discord:
    allowed_tools: ["web_search", "browser_automation"]   # Discord 限制更严格

### 

#### 

#### 

#### 

### 
本章小结  连接外部平台后，Hermes 才真正从“工具”变成了“伴侣”。你现在可以在手机上随时指挥它工作，而它则在云服务器上 24 小时为你服务。

## 

Hermes 之所以强大，很大程度上是因为它拥有47+ 内置工具，并且支持通过 MCP 协议无限扩展。这些工具让 Hermes 不再只是“聊天机器人”，而是真正能执行真实世界任务的代理。
本章将带你全面掌握工具的使用、配置与扩展。

### 
Hermes 的工具大致分为以下几大类：
查看当前所有可用工具：
hermes tools list
在对话中输入：
/tools
Hermes 会以清晰表格形式列出所有工具及其当前状态。

### 
这是最常用的工具管理命令：
hermes tools
运行后会进入交互式菜单，你可以：
推荐配置策略（新手建议）：
命令行快速配置示例：
# 全局禁用某个工具
hermes config set tools.disabled terminal,bash

# 为 Telegram 单独配置
hermes config set gateway.telegram.allowed_tools web_search,browser_automation,vision

### 
Hermes 支持通过 Python 快速注册自定义工具，非常强大。

#### 
mkdir -p ~/.hermes/tools
nano ~/.hermes/tools/my_tools.py
from hermes.tools import tool
@tool(
    name="get_weather",
    description="获取指定城市的实时天气信息",
    parameters={
        "city": {"type": "string", "description": "城市名称，如 Beijing"}
    }
)
def get_weather(city: str) -> str:
    # 这里可以调用真实天气 API
    import requests
    # 伪代码示例
    return f"{city} 今天晴朗，气温 22°C"
hermes tools reload
之后在对话中就可以直接说：
帮我查一下北京今天的天气
Hermes 会自动调用你刚注册的工具。

### 
MCP 是 Hermes 扩展工具生态的核心协议。通过 MCP，你可以连接：

#### 
编辑 ~/.hermes/config.yaml：
mcp_servers:
  github:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: "ghp_xxxxxxxxxxxx"
  
  postgres:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-postgres"]
    env:
      DATABASE_URL: "postgresql://user:pass@localhost:5432/mydb"
添加后重启网关或执行：
hermes mcp reload
现在 Hermes 就可以直接操作你的 GitHub 仓库或数据库了！

### 

#### 

#### 

### 
本章小结  工具系统是 Hermes 的“双手”。掌握工具配置和扩展后，你可以让 Hermes 完成几乎任何可自动化的任务。

## 

这是 Hermes Agent 最核心、最具革命性的功能——技能系统（Skills System）。
其他 AI Agent 主要靠人工写 Prompt 或固定工作流，而 Hermes 能从真实使用经验中自动提炼技能，并在后续任务中持续优化。这就是它“越用越聪明”的根本原因。

### 
技能（Skill） 是一个结构化的 Markdown 文件，存储在 ~/.hermes/skills/ 目录下。它包含：
与传统 Prompt 的本质区别：
简单说：Prompt 是“一次性说明书”，技能是“可进化的操作手册”。

### 

#### 
# 搜索社区技能库
hermes skills search kubernetes

# 安装技能
hermes skills install openai/skills/k8s-deployment

# 查看已安装技能
hermes skills list

# 查看技能详情
hermes skills show k8s-deployment

# 删除技能
hermes skills remove k8s-deployment

# 更新所有技能
hermes skills update

#### 
输入：
/skills
Hermes 会列出当前可用技能，并支持直接调用。

### 
这是 Hermes 最神奇的部分。完整流程如下：
查看自动生成的技能：
ls ~/.hermes/skills/
每个技能文件都包含清晰的 YAML frontmatter 和结构化内容，方便人工阅读和修改。

### 
虽然 Hermes 能自动生成技能，但手动创建高质量技能仍然非常有价值。

#### 
nano ~/.hermes/skills/my-custom-skill.md
---
name: my-custom-skill
description: 快速部署带 SSL 的 Nginx + Node.js 项目
version: 1.0.0
tags: [deploy, nginx, nodejs, ssl]
---

# 任务目标
快速在一台干净的 Ubuntu 服务器上部署带 Let's Encrypt SSL 的 Node.js 项目。

# 前置条件
- 服务器已安装 Docker
- 域名已指向服务器 IP
- 拥有 Cloudflare 或阿里云的 API Key（可选）

# 执行步骤
1. 使用 terminal 工具克隆项目代码
2. 运行 docker-compose up -d
3. 配置 Nginx 反向代理
4. 使用 certbot 获取 SSL 证书
5. 设置自动续期

# 常见错误处理
- 端口冲突 → 检查 80/443 端口占用
- SSL 申请失败 → 检查 DNS 解析和防火墙

# 成功判据
- 浏览器访问域名返回 200
- HTTPS 证书有效
hermes skills reload

### 
Hermes 官方维护了 agentskills.io 平台，专门用于分享和发现高质量技能。
发布自己的技能：
hermes skills publish my-custom-skill
搜索并安装社区技能：
hermes skills search "deploy fastapi"
hermes skills install community/fastapi-production
目前社区已经积累了大量实用技能，覆盖 DevOps、内容创作、数据分析、个人自动化等场景。

### 
查看技能版本历史：
hermes skills history my-custom-skill

### 
本章小结  技能系统是 Hermes “自我进化”的引擎。掌握它之后，你会真正感受到 Hermes 正在随着你的使用不断成长。

## 

恭喜你来到本教程的进阶部分！在前八章中，你已经掌握了 Hermes 的核心能力。从本章开始，我们将解锁一系列高级功能，让 Hermes 真正成为你的全能数字助手。

### 
Hermes 支持实时语音交互，可在 CLI、Telegram、Discord 等平台使用。

#### 
# 安装语音依赖（首次使用）
pip install "hermes-agent[voice]"

# 开启语音
hermes config set voice.enabled true
在对话中输入：
/voice on
使用方法：
实用场景：

### 
这是 Hermes 最实用的高级功能之一。你可以用自然语言设置定时任务。

#### 
在对话中说：
每天早上 8 点给我发送昨日项目进度 + 今日待办 + 相关技术文章摘要
Hermes 会自动解析并创建 cron 任务。你也可以更精确地描述：
每周一上午 9 点执行代码仓库备份，并把结果发送到我的邮箱

#### 
# 查看所有定时任务
hermes schedule list

# 删除任务
hermes schedule remove <task_id>

# 手动触发任务
hermes schedule run <task_id>
常见自动化场景：

### 
Hermes 支持多种人格切换，并可通过 SOUL.md 进行深度定制。

#### 
/personality pirate          # 海盗风格
/personality professional    # 专业严谨
/personality teacher         # 耐心教师
/personality concise         # 极简风格

#### 
创建或编辑 ~/.hermes/SOUL.md：
# Hermes 的灵魂设定

你是一个拥有 10 年经验的资深全栈工程师 + 产品经理。
你的风格：专业、严谨、略带幽默、注重长期价值。
你总是先理解用户真实意图，再给出最优方案。
你会主动提醒用户潜在风险和长期影响。
你特别擅长把复杂技术问题用简单语言解释清楚。
保存后所有对话都会遵循这个设定，直到你修改它。

### 
Hermes 支持为特定项目创建持久化上下文文件。

#### 
在项目根目录创建 .hermes-context.md：
# 项目上下文

## 项目名称
Hermes Agent 教程系列

## 技术栈
- Python 3.11+
- FastAPI + SQLAlchemy 2.0
- PostgreSQL + Redis
- Docker + Docker Compose

## 开发规范
- 所有 API 必须有类型注解
- 提交前必须通过 lint 和测试
- 文档优先（先写 README 再写代码）

## 当前阶段
正在撰写第 9 章高级功能教程
Hermes 在对话中检测到当前目录有该文件时，会自动加载并参考其中的内容。
这让 Hermes 在处理不同项目时能保持一致的“项目记忆”。

### 
Hermes 支持与主流编辑器深度集成（实验性功能）。

#### 
pip install -e '.[acp]'
hermes acp
启动后，你可以在 VS Code / Cursor / Zed 等编辑器中直接调用 Hermes 进行：
这让 Hermes 成为真正的“IDE 级 AI 助手”。

### 
Hermes 内置了强大的研究功能，适合需要深度分析和长期优化的场景。

#### 
这些功能让 Hermes 不仅是一个生产力工具，更是一个可研究、可进化的智能体平台。

### 
本章小结  掌握这些高级功能后，Hermes 已经从“聊天工具”进化成了真正的全能数字员工。接下来我们将讨论如何把 Hermes 部署到生产环境，让它真正 24/7 稳定运行。

## 

到目前为止，你已经在本地或开发环境熟练使用了 Hermes。现在是时候把它部署到生产环境，让它真正成为24/7 在线、不间断工作的数字员工了。
本章将系统讲解 Hermes 的多种部署方式，并给出生产级运维的最佳实践。

### 

#### 
推荐场景：日常开发、学习、短期任务。

#### 
目前社区最受欢迎的方案是低配 VPS + Docker 沙箱，月成本约 3–6 美元即可实现 24/7 运行。
推荐平台与一键模板（2026 年 4 月）：
部署步骤（以 Hostinger 为例）：
一键安装命令（手动部署）：
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
hermes config set terminal.backend docker

### 
这是目前最稳定、最推荐的生产部署方式。

#### 
# 拉取最新镜像
docker pull nousresearch/hermes-agent:latest

# 运行容器
docker run -d \
  --name hermes \
  -v ~/.hermes:/root/.hermes \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --restart unless-stopped \
  nousresearch/hermes-agent:latest

#### 
创建 docker-compose.yml：
version: '3.8'

services:
  hermes:
    image: nousresearch/hermes-agent:latest
    container_name: hermes
    restart: unless-stopped
    volumes:
      - ~/.hermes:/root/.hermes
      - /var/run/docker.sock:/var/run/docker.sock
      - ./projects:/root/projects          # 挂载你的项目目录
    environment:
      - HERMES_ENV=production
    ports:
      - "8080:8080"   # 如需暴露 API
启动：
docker compose up -d
优势：

### 
如果你希望完全按需付费，可以选择 Serverless 平台：
Modal 部署示例：
pip install modal
modal deploy hermes_modal.py
这些方案特别适合偶尔使用但需要高可用的场景。

### 

#### 
Hermes 的核心数据都保存在 ~/.hermes/ 目录下，建议定期备份：
# 备份脚本示例
tar -czf hermes-backup-$(date +%Y%m%d).tar.gz ~/.hermes
推荐备份内容：

#### 

### 

#### 
hermes config set security.require_approval true
开启后，所有危险命令（删除文件、执行脚本等）都会先征求你的确认。

#### 
始终使用 Docker 后端：
hermes config set terminal.backend docker

#### 

#### 

#### 
开启详细审计日志：
hermes config set logging.level audit

### 
本章小结  部署完成后，Hermes 就真正从“实验玩具”变成了生产级数字员工。接下来我们将讨论如何长期高效、安全地使用它。

## 
恭喜你来到本教程的倒数第二章！在前十章中，你已经从零开始掌握了 Hermes 的安装、配置、核心机制、高级功能和部署方式。
本章将汇总最实用、最容易被忽视的经验和坑，帮助你少走弯路、长期高效使用 Hermes。

### 
Hermes 的自改进能力依赖于高质量的交互。以下技巧能显著提升效果：

#### 

### 

#### 

#### 

### 
Hermes 的成本主要来自模型调用和工具使用。以下是实用策略：
推荐配置：
hermes config set routing.enabled true
hermes config set tools.max_concurrent 2

### 
遇到问题时，永远先运行：
hermes doctor
以下是社区最常见的 15 个问题及解决方案：

### 

#### 

#### 
最佳实践：

### 
本章小结  掌握这些最佳实践后，你已经具备了长期稳定、高效使用 Hermes 的能力。接下来我们将通过真实项目案例，把所有知识串联起来。

## 

恭喜你来到本教程的最终章！
在前 11 章中，你已经系统掌握了 Hermes 的全部核心能力。现在是时候把这些知识转化为真实生产力了。
本章将通过 5 个完整实战项目，带你亲手构建属于自己的智能代理系统。每个案例都包含完整配置、关键提示词和可复用技能。

### 
目标：每天早上自动为你抓取最新 AI/技术论文，生成中文摘要，并同步到 Notion 数据库。

#### 
预期效果：每天 7:30 准时收到 Telegram 消息 + Notion 自动更新，无需任何手动操作。

### 
目标：让 Hermes 成为你的 24/7 DevOps 工程师，能自动处理代码审查、部署和线上问题排查。

#### 
闭环效果：从代码提交 → 自动 review → 一键部署 → 异常自动修复，形成完整闭环。

### 
目标：让 Hermes 帮你完成从选题到多平台发布的全流程内容生产。

#### 
成果：从选题到多平台发布，全程仅需你确认最终文案，效率提升 5-8 倍。

### 
目标：打造一个家庭/团队的智能管家。

#### 

### 
目标：让 Hermes 成为你某个长期项目的专属助手，拥有完整项目记忆和专属技能库。

#### 
长期价值：半年后，Hermes 对这个项目的理解深度可能超过你自己，能主动提出优化建议和风险预警。

### 

## 
恭喜你完成了 《Hermes Agent 详细教程》 全 12 章！
你现在已经拥有了：
下一步建议：
最后的话：
Hermes Agent 不是终点，而是一个起点。它代表了一种全新的 AI 使用范式——从“使用工具”到“培养伙伴”。
希望这个教程能帮助你真正把 Hermes 变成你最得力的数字员工。
如果你在实践过程中遇到任何问题，随时回来翻阅本教程，或直接在 Discord 社区提问。
现在，开启你的 Hermes 之旅吧！
教程后续还会持续更新，记得关注
更多ai的使用场景以及使用技巧，记得关注我v：ddsh2046

[多维表格 - 请到飞书网页端查看]