# 这两个开源利器，OpenCLI和BB-Browser，让你的OpenClaw瞬间拥有“全互联网”能力

**公众号**: 未找到公众号
**作者**: 未找到作者
**发布时间**: 2026年3月16日 07:20

---

大家好，我是你们的AI工具猎手夹克。
在浏览器和命令行之间反复切换，已成为无数爬虫人的痛点。更别提把网页数据喂给 AI Agent 时，还要手写爬虫、处理反爬、维护 Selector……今天要说的这两个项目，正是可以解决这“最后一公里”的黑科技工具。
无需API密钥、无需爬虫、无需反爬风险，用真实登录态，28+命令或者103命令，全网数据一键CLI调用。搭配OpenClaw，更可以让本地AI代理瞬间拥有“全互联网”能力。
今天这篇文章不聊大模型参数，不聊框架选型，就聊聊这两个刚刷新的开源项目，它们把“浏览器即API”这个概念真正落地了：
•
OpenCLI
：将任意网站变成CLI的AI原生工具，已支持16个站点28+命令。
•
BB-Browser
：浏览器控制中枢 + MCP服务器，103个命令覆盖36个平台，还内置OpenClaw适配模式。
希望你看完下面的内容，可以眼前一亮。
一、OpenCLI：AI驱动的“网站CLI化”神器
一句话定位
：它把任何网站（B站、知乎、小红书、Twitter、GitHub、Yahoo Finance…）直接变成命令行工具，复用你Chrome的已登录状态，账号安全、数据真实。
核心亮点
（来自官方README）：
• 账号安全：
绝不触碰你的密码
，直接复用Chrome Session。
• AI原生：
explore
、
synthesize
、
generate
、
cascade
四连招，AI自动发现API、生成适配器。
• 声明式YAML + TypeScript逃逸：大部分适配器30行YAML搞定，复杂场景（GraphQL、XHR劫持）直接写TS。
• 输出灵活：table（默认美观表格）、json、md、csv、verbose调试模式。
开源网址：
https://github.com/jackwener/opencli
安装（3秒搞定）
：
npm install -g @jackwener/opencli
基础使用示例
：
# 列出所有命令
opencli list
# 无需浏览器（纯公共API）
opencli hackernews top --
limit
5
# 需要浏览器（复用登录态，必须先在Chrome登录目标站）
opencli bilibili hot --
limit
10 -f json
opencli zhihu hot -f table
opencli xiaohongshu search
"AI代理"
--
limit
5
AI工作流（最惊艳的部分）
：
opencli explore https://example.com --site mysite
# AI自动探测
opencli synthesize mysite
# 生成YAML适配器
opencli generate https://new.site --goal
"hot"
# 一键生成
理性分析
：
优点
：极简、轻量、AI加持下扩展性爆炸。比自己写Playwright脚本快10倍，数据直接来自真实用户视角，防反爬效果拉满。
缺点
：浏览器命令依赖Chrome + Playwright MCP Bridge扩展，配置稍繁琐（token设置容易忘）。适配器依赖网站结构，改版就可能失效（虽有AI辅助，但仍需人工干预）。目前仅180多星，社区小，issue响应全靠作者一人。资源占用待优化，后台Chrome常驻，吃内存。
使用建议
：
• 个人自动化/数据采集首选，生产环境慎用（建议加定时任务+重试机制）。
• 先用公共API命令练手，再上浏览器命令。
• 定期
npm update -g @jackwener/opencli
，作者昨天刚发v0.2.0，更新极快。
• 安全建议：单独开一个Chrome Profile专门给OpenCLI用，别混日常账号（非常重要）。
二、BB-Browser：把浏览器彻底变成AI的“超级API”
一句话定位
：你的浏览器就是API。CLI + MCP服务器，让AI代理（Claude Code、Cursor等）直接操控真实Chrome，执行103个站点命令，还支持OpenClaw无扩展模式。
核心亮点
：
• 103命令 × 36平台（Twitter、Reddit、知乎、BOSS直聘、雪球、Arxiv、YouTube转录…几乎全覆盖）。
• 支持
--json
+
--jq
过滤、
--tab <id>
多标签并发。
• 内置浏览器基础操作：
open
、
click
、
fill
、
eval
、
fetch
、
screenshot
。
•
OpenClaw模式
：
--openclaw
一键切换，无需安装Chrome扩展。
• MCP服务器模式：无缝接入Claude Code/Cursor，让AI直接“上网”。
开源网址：
https://github.com/epiral/bb-browser
安装
：
npm install -g bb-browser
实战命令
：
# 更新社区适配器（作者维护+社区贡献）
bb-browser site update
# 搜索示例
bb-browser site twitter/search
"AI agent"
bb-browser site boss/search
"AI工程师"
--json
bb-browser site xueqiu/hot-stock 5 --jq
'.items[] | {name, changePercent}'
# OpenClaw模式（无需扩展）
bb-browser site reddit/hot --openclaw
# 浏览器基础操控
bb-browser open https://github.com
bb-browser screenshot
bb-browser
eval
"document.title"
MCP配置（接入AI代理）
： 在ClaudeCode/Cursor的mcpServers里加一段即可，AI就能调用整个浏览器。
理性分析
：
优点
：真正实现了“浏览器即API”。比传统Playwright/Selenium高效，AI代理的世界从“文件+终端+几个API”直接升级成“全互联网”。
缺点
：依赖Daemon（默认localhost:19824），开0.0.0.0远程访问要慎重（安全风险）。适配器复杂度分级：简单Cookie 1分钟，复杂Webpack/Pinia要10分钟，网站大改版仍需维护。非无头，Chrome常驻，吃CPU/内存；多代理并行时容易卡。330星虽比OpenCLI多，但仍属小众项目，长期维护依赖社区。
使用建议
：
• AI代理开发者必备，直接当MCP技能用。
• 日常数据采集推荐OpenClaw模式，省扩展安装。
• 生产场景：加
--host 127.0.0.1
限制访问 + 日志监控。
• 测试新适配器时用
bb-browser site info xxx
先查结构。
三、与OpenClaw的使用场景深度结合
这可能是这类项目
最被低估的价值
，它不是“替代浏览器”，而是
让AI Agent拥有网页超能力
。比如现在爆火的龙虾，OpenClaw作为自托管个人 AI 助手，支持 50+ 集成、CLI 工具调用、技能注册（skills registry），已原生支持把外部 CLI（如 Box CLI）注册为 Agent可调用的工具。
与他们正好完美契合
，比如以OpenCLI为例：
1.
数据采集与实时监控闭环
OpenClaw 不再需要写自定义浏览器技能，可以直接注册 OpenCLI 命令：
• 每日自动拉取 Bilibili 热榜 + 知乎热搜 → JSON 喂给 Agent 分析趋势 → 生成总结推送 Telegram/WhatsApp。
• 示例工作流：在 OpenClaw skills 中添加
opencli bilibili hot -f json
，Agent 用自然语言指令“帮我查今天 B 站 AI 视频热榜并总结 top3”即可执行，无需人工干预。
2.
跨站自动化决策链
结合 OpenClaw 的多模型支持（Claude、GPT、Grok）和 cron/sessions：
• Agent 先用
opencli github trending -f json
获取趋势，再用
opencli zhihu search "XXX" -f json
查中文讨论，最后决策“是否要 Fork 这个 repo 并 PR”。
• 真实场景：内容创作者的 OpenClaw 每天自动抓小红书/YouTube 热度，生成选题报告；开发者 Agent 监控 Reuters + Yahoo Finance 新闻，触发交易提醒。
3.
AI 自助适配新网站（元能力）
OpenCLI 自身就是 Agent：
• OpenClaw 可调用
opencli explore + synthesize + generate
，让自身“学会”一个新网站（例如刚上线的某平台），生成适配器后可以永久使用。
• 这形成
自进化循环
：OpenClaw 不再受限于预置工具，能自主扩展网页能力。
4.
与 LangChain 等框架的无缝互补
OpenCLI 命令可直接作为 LangChain Tool：
# LangChain 示例（伪码）
tools = [Tool.from_function(opencli_bilibili_hot, ...)]
agent = create_react_agent(llm, tools)
其实OpenClaw 用户几乎是最简单的：CLI 技能注册即用，无需写 Python。
实战效率
：根据社区反馈，OpenClaw接入OpenCLI后，Agent 的“网页操作成功率”从 40% 跳到 95%，因为避开了 Selenium 风控和不稳定 Selector，全部走官方会话。
OpenCLI/BB-Browser与你的OpenClaw结合，可以让你真正拥有一个掌握全网的超级AI助手。
四、思考：便利背后的风险与边界
我必须说两句“泼冷水”的话：
1.
隐私与安全
：真实登录态是双刃剑。浏览器被AI控制，万一代理被劫持，后果比传统爬虫严重得多。建议：专用Chrome Profile + 最小权限Daemon + 定期审查技能。
2.
稳定性
：网站改版 = 适配器失效。AI自动生成虽酷，但准确率并非100%。别把核心业务全押上去。
3.
资源与合规
：常驻Chrome吃资源；部分站点（尤其是金融、招聘）明确禁止自动化，法律风险自担。
4.
社区成熟度
：两个项目更新都极快（昨天还刚发新版），但星数仍不高，长期看需要更多贡献者。
底线建议
：个人玩、学习、辅助工作极度推荐；涉及金钱、敏感数据、对外服务，建议加人工审核层或回退到官方API。
结语：浏览器时代的AI新范式已来
OpenCLI和BB-Browser真正把“浏览器即API”从概念变成了可落地的命令行工具。搭配OpenClaw，你不再需要为每个网站写爬虫，不再被API限额卡脖子，不再担心反爬，你只需要说一句自然语言，AI就替你干，就是这么简单。
觉得有用吗，点赞、转发给同样被爬虫折磨的同学吧。让技术永远服务于人，保持理性地拥抱，才是真正聪明的使用方式。我们下期再见，祝你周末愉快，AI永不叛变。🦞
（本文基于官方GitHub、社区反馈、项目测试，观点纯个人中立分析，不构成投资建议。如有更新，以GitHub最新为准。）
