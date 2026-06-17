---
title: "魔改源码｜Claude Code 桌面端，功能直接拉满"
source: "海鱼星的荷花塘"
source_url: "https://mp.weixin.qq.com/s/PeJFf68XaqT0srNfWguPYw"
date: "2026-04-27"
tags: [Claude Code, cc-haha, 桌面端, Computer Use, 飞书集成]
---

Claude Code 官方桌面端可以使用第三方 API、不需要登录，但有些功能是缺失的，比如 **Computer Use**（让 AI 直接操作电脑）就没有。用官方桌面端感觉没什么用。

在逛 GitHub 的时候，发现了一个基于泄露的 Claude Code 源代码做成的桌面端项目——**cc-haha**。

它是基于 Claude Code 泄露源码修复的本地可运行版本，支持接入任意 Anthropic 兼容 API（MiniMax、OpenRouter 等）。在完整 TUI 之外，还补全了 Computer Use（macOS / Windows）、打造了图形化桌面端，并支持通过 Telegram / 飞书完整远程驱动。

```
https://github.com/NanmiCoder/cc-haha
```

![封面图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_001.png)

![GitHub 项目主页截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_002.png)

## 下载与安装

点击 **Releases** 下的最新版。

![Releases 页面截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_003.png)

选择你的系统版本（本文以 Windows 版本为例）。

![系统版本选择截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_004.png)

双击下载的安装包，默认安装即可。

![安装过程截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_005.png)

安装后自动打开，左侧会显示之前的历史记录。

![历史记录截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_006.png)

## 配置模型

### 添加服务商

点击 **设置** → **服务商** → **添加服务商**。

![设置界面截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_007.png)

可以看到国内主流的服务商都支持，还支持自定义。

![服务商列表截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_008.png)

### 以接入 DeepSeek V4 系列模型为例

打开 [DeepSeek 官网](https://platform.deepseek.com/)，选择 API 开放平台。

![DeepSeek 官网截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_009.png)

使用微信登录。

![微信登录截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_010.png)

登录后，点击左侧的 **API Keys**，点击 **创建 API Key**。

![API Keys 页面截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_011.png)

创建一个 API Key。

![创建 API Key 截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_012.png)

回到 cc-haha 的添加服务商界面，输入刚才申请的 API Key 和 DeepSeek 模型：`deepseek-v4-flash`、`deepseek-v4-pro`，点击添加。

![配置模型截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_013.png)

添加后，回到服务商页，在 DeepSeek 服务商处点击 **设为默认**，完成模型配置。

![设为默认截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_014.png)

## Computer Use 功能

再来看左侧的 **Computer Use**，需要先安装虚拟环境和依赖包，点击 **安装环境**。

![Computer Use 安装环境截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_015.png)

环境安装完成：

![环境安装完成截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_016.png)

### 测试 Computer Use

点击 **新建任务**，选择项目文件夹。

![新建任务截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_017.png)

点击权限，给 Claude Code 所有权限。

![权限设置截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_018.png)

选择模型（此处使用了 MiniMax 2.7；最初尝试 DeepSeek 提示 API 无效，可能是刚出的模型尚未适配）。

![选择模型截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_019.png)

告诉 Claude Code：

```
使用 Computer Use 打开 Google Chrome，找到地址栏，输入 https://www.bilibili.com，让我访问 B 站。
```

![Computer Use 执行截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_020.png)

开始操作电脑，成功打开了 B 站：

![B 站访问结果截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_021.png)

![Computer Use 操作过程截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_022.png)

![Computer Use 操作界面截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_023.png)

## 定时任务

点击左侧的 **定时任务**，可以设置定时任务。

![定时任务设置截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_024.png)

## 技能管理

点击 **设置** → **技能**，可以看到已安装的技能。目前尚未接入技能市场，需要手动安装。

![技能管理截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_025.png)

## IM 接入

点击 **IM 接入**，可以看到支持飞书和 Telegram。

![IM 接入截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_026.png)

### 飞书机器人配置

直接点击下面链接创建（飞书官方已为 OpenClaw 配置好了各种权限的机器人）：

```
https://open.feishu.cn/page/openclaw?form=multiAgent
```

输入助手名，点击 **立即创建**。

![创建飞书助手截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_027.png)

继续创建。

![继续创建截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_028.png)

把 **App ID** 和 **App Secret** 保存下来，开始下一步的机器人菜单配置。

![保存 App ID 和 Secret 截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_029.png)

进入飞书开发者后台，选择刚创建的机器人：

```
https://open.feishu.cn/app?lang=zh-CN
```

![飞书开发者后台截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_030.png)

双击进入刚创建的助手，点击左侧的 **机器人**。

![机器人配置页面截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_031.png)

点击 **机器人自定义菜单**。

![自定义菜单截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_032.png)

找到 **机器人自定义菜单** 下的 **菜单状态**，点击开启，展示形式选择 **可切换菜单**。

![菜单状态设置截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_033.png)

默认展示里选择 **输入框**。

![输入框设置截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_034.png)

在菜单配置里，名称输入 `/projects`，相应动作选择 **发送文字消息**。

![配置 /projects 菜单截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_035.png)

同样配置 `/clear`、`/new`，点击保存。

![配置 /clear /new 截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_036.png)

保存后提示创建新版本，在最下面点击保存。

![创建新版本截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_037.png)

点击 **确认发布**。

![确认发布截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_038.png)

回到 cc-haha 的 IM 配置页面，输入 App ID 和 App Secret。

![输入 App ID 和 Secret 截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_039.png)

点击 **生成配对码**。

![生成配对码截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_040.png)

点击 **保存**，给助手发消息提示没有配对。

![配对提示截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_041.png)

发送配对码，配对成功，显示历史项目。

![配对成功截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_042.png)

回到配对界面，可以看到已配对用户。至此，就完成了用飞书控制 Claude Code 的配置。

![已配对用户截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_043.png)

![最终效果截图](../assets/2026-04-27-魔改源码ClaudeCode桌面端/img_044.png)

好了，今天就分享到这里，希望对大家有所帮助，有问题可以和我在评论区讨论～
