# AI 自动化真牛X，开源 BrowserAct Skills 让我彻底解放

> **来源：** 公众号「小华同学ai」@小华
> **发布时间：** 2026-06-22 02:09
> **原文：** https://mp.weixin.qq.com/s/MCVl9OO7mUglXYD_OvBz5w
> **GitHub：** https://github.com/browser-act/skills
> **License：** MIT

## 概述

BrowserAct Skills 是一个专为 AI Agent 打造的浏览器自动化 CLI 工具，提供三层反爬体系、人机协作、并行隔离、索引交互等能力。附带 30+ 预构建 Skill 覆盖电商/社媒/视频/搜索等主流平台，以及 Skill Forge 可自动生成可复用的爬虫 Skill 包。

---

## 三层反爬体系

```
环境层（伪装真实用户）→ 执行层（自动解决验证）→ 人类层（远程人工接管）
```

### 第一层：环境层
| 手段 | 作用 |
|------|------|
| 指纹伪装 | Canvas/WebGL/字体/插件统一伪造 |
| Navigator 修补 | webdriver/chrome.runtime 等检测标记归一化 |
| TLS 指纹轮换 | 匹配真实浏览器签名特征 |
| 代理系统 | 动态轮换IP / 固定托管IP / BYO |
| 隐私模式 | 每个会话全新指纹 + 空 Profile |

### 第二层：执行层
```bash
# 一键提取受保护页面（零配置）
browser-act stealth-extract https://protected-site.com

# 自动识别并解决验证码
browser-act --session s1 solve-captcha
```

### 第三层：人类层
```bash
# 生成远程接管链接，任何设备打开即可操控
browser-act --session my-task remote-assist --objective "完成双重认证"
```
传统方案需 VNC/RDP 且必须同机，BrowserAct 直接生成 URL，任何人任何设备可远程接管，操作完成后 Agent 自动继续。

---

## 三种浏览器模式

| 模式 | 场景 | 特点 |
|------|------|------|
| `chrome`（Profile 导入） | 复用 Chrome 登录态 | 导入 cookies/localStorage，隔离运行 |
| `chrome-direct`（CDP 直连） | 需要扩展/证书/SSO | 零配置直连本地 Chrome |
| `stealth` 隐私模式 | 无需登录批量抓取 | 每次全新指纹+动态IP，零残留 |
| `stealth` 固定身份 | 多账号并行运营 | 稳定指纹+固定IP，不被关联 |

---

## LLM 友好设计

### 紧凑文本输出（节省 Token）
```
url=https://example.com/login
title=Login
*[1]<div id=login-form />
*[2]<input type=email placeholder=Email />
*[3]<input type=password placeholder=Password />
*[4]<button id=submit /> Sign In
```

### 索引交互（无需 XPath/CSS 选择器）
```bash
browser-act --session s1 state         # 返回可交互元素列表
browser-act --session s1 click 3       # 按索引点击
browser-act --session s1 input 2 "hi"  # 按索引输入
```

### 语义记忆
每个浏览器实例带 `desc` 字段（自然语言描述用途），Agent 根据任务含义匹配浏览器，无需硬编码 ID。

---

## Skill Forge：自动生成爬虫 Skill

四步工作流：**描述需求 → 自动探索 → 生成 Skill 包 → 自测修复**

- API 优先（抓网络请求发现端点，稳定度 10x），DOM 降级
- 业务变量变 CLI 参数，产出 SKILL.md + 可执行脚本
- 子 Agent 自动验证+自动修复

```bash
forged-skill linkedin-jobs --keyword "AI Engineer" --location "Remote"
```

---

## 30+ 预构建 Skill 分类

| 分类 | 数量 | 示例 |
|------|------|------|
| 电商 | 20 | 淘宝搜索/详情、闲鱼、Amazon ASIN/评论、通用电商 |
| 线索挖掘 | 11 | Google Maps 商家/评论、LinkedIn/Indeed 岗位、GitHub 贡献者、Product Hunt |
| 搜索与研究 | 5 | Google SERP/News、网页 Markdown 提取、多源研究助手 |
| 社交监听 | 21 | 小红书搜索/详情/自动发文、微信公众号、知乎、X/Twitter、Instagram、Facebook、Reddit |
| 视频平台 | 14 | YouTube 搜索/字幕/评论/达人、TikTok 搜索/主页 |

---

## 安全：确认门控 + 全本地化

- 创建/删除/导入浏览器、修改代理、切换隐私模式等敏感操作必须用户批准
- Cookies、会话、页面内容、截图、Profile 全部本地存储，不离开本机
- 唯一例外：`solve-captcha` 仅将验证码图片发云端求解

## 快速上手

```bash
# 安装
Install browser-act. Skill source: https://github.com/browser-act/skills/tree/main/browser-act

# 提取受保护页面
browser-act stealth-extract https://example.com

# 完整自动化
browser-act --session my-task browser open <id> https://example.com
browser-act --session my-task state
browser-act --session my-task click 3
```

兼容 Windows/macOS/Linux，支持 Claude Code / Cursor / OpenCode / OpenClaw / Codex / Gemini CLI 等。

## 定价

- 浏览器自动化、Chrome/Chrome-direct：免费（无需注册）
- Stealth 浏览器（≤5 个）、stealth-extract、solve-captcha、remote-assist、Skill Forge：免费（登录）
- Stealth 浏览器 >5 个、托管代理：付费

## 归档日志

- 2026-06-22 归档
