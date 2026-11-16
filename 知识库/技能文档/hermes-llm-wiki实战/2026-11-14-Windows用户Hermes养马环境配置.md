---
title: "windows用户大气Hermes小白从零配置出最舒服的养马环境，养马体验提升2-3倍，几乎完全告别指令~"
source: "https://zcnv48kqce5a.feishu.cn/wiki/QP15wSI24iLyymkL3HgcSLLHnvh"
source_platform: "飞书文档"
archived: 2026-11-14
tags: [Hermes, Windows, WSL, 环境配置, 飞书归档]
description: "Windows 用户从零配置最舒服的 Hermes 养马环境，体验提升2-3倍，几乎完全告别指令"
series: "Hermes&OpenClaw实战进阶配套笔记"
---

# windows用户大气Hermes小白从零配置出最舒服的养马环境，养马体验提升2-3倍，几乎完全告别指令~

> 来源：飞书文档（李君陌AI笔记）｜Hermes & OpenClaw 实战进阶配套笔记 第5篇
> 原文：https://zcnv48kqce5a.feishu.cn/wiki/QP15wSI24iLyymkL3HgcSLLHnvh
> 归档时间：2026-11-14

hermes终于出了windows原生安装的方式了，比起WSL安装真的方便很多，今天我们分别用windws原生和WSL两种方式，分享一个新手从零开始完成一个非常舒服的Hermes Agent养马环境的搭建，几乎可以告别指令，各个方面都可以进行可视化操作了。除了基础的包括TUI直接使用hermes以及通过飞书来进行交互，还搭建了可以汇总各个通道并且可以直接发消息的webui，对不熟悉linux的也安装了宝塔面板方便来进行文件修改，然后还给Hermes实现了一个每天自动更新的可视化进化面板，可以方便看到每一天进化的技能、记忆等内容，最后给WSL也提供了一个dashboard，可以方便备份还原你的整个Hermes环境，可以踏踏实实的养虾，也能轻松的将你的整套hermes环境移植到另外一台电脑，这一套整完，你的养马环境真的就很舒服了。

> `first_place_medal`
> **1.从WSL—>Ubuntu—>Hermes Agent**
> **2.配置好飞书**
> **3.安装好第三方WebUI，通过web管理和操作Hermes**
> **4.安装小白也可以舒服操作Linux的宝塔面板**
> **5.搭建可视化进化面板：让每一天的进化都能看见**
> **6.WSL也可以可视化操作，并且方便备份Hermes环境**

## 视频讲解

[iframe:https%3A%2F%2Fwww.bilibili.com%2Fvideo%2FBV1zn576XEb3]

## **方法一：windows原生安装**

https://github.com/NousResearch/hermes-agent/tree/main

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/K9TfbdSrOoXDq8xqT15c9ZxEnDe.png)

windows原生安装，目前属于测试版，还没经过大量测试，可能不太稳定，安装确实非常简单，直接打开终端管理员输入以下指令即可。

```plaintext
irm https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1 | iex
```

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/UJ78bfdRPoH4cZxQpHqcA6mKnFc.png)

大概整个过程10分钟左右，安装完了如果你装过openclaw会被识别到，然后可以选择导入openclaw养虾的信息，确定之后进入下一步，然后剩下的步骤直接跳到本期文档**四、hermes的安装和使用**的4.**2.配置大模型**，前面的过程都可以跳过

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/AahcbSvB2oXfHAxCZzbchnqRnyc.png)

完成大模型和飞书的配置后，就可以正常使用了。

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/DNMtbarfmoG5WTxaikKcIP30n5b.png)

windows原装安装的好处是不用再去熟悉linux了，包括文件编辑都可以在windows里操作，而且跟windows的交互会更好一些。

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/FRunbUbmGo4LiFxWKHecfXgqn0k.png)

## 方法二：WSL安装

## **一、启动并安装wsl**

先输入wsl看一下是否安装，如果没安装会自动触发下载Linux的windows子系统

```plaintext
wsl
```

如果没自动触发，就手动输入以下命令来触发安装

```plaintext
wsl --install
```

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/OL7ib2eWho9UhsxvJwGchvSAndg.png)

安装完成后需要重启电脑

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/CzEUbWTS9obbWNxbZYicmASfnmN.png)

**安装完成后要重启电脑**

## **二、安装ubuntu**

选 Ubuntu-22.04 是目前**最稳妥、兼容性最好、教程最多**的选择

```plaintext
wsl --install -d Ubuntu-22.04
```

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/UXPYbmTbjooaBNxdVZAcLAlEnZc.png)

安装完了之后会提示输入用户名和密码，密码这里是不会显示出来的，输好了回车就可以

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/ONtUbvyqroSJS6x8atPcW2avnzd.png)

输好了，当你看到这个绿色+蓝色的这个行命令行，就说明已经安装好ubuntu了

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/HNVsb8ShQoVd59xFDntcX4QUnog.png)

## **三、前置环境准备工作**

### 3.1.更改加速源以及更新软件包索引

**换成加速的源**

```bash
sudo sed -i 's|http://archive.ubuntu.com|https://mirrors.aliyun.com|g' /etc/apt/sources.list
sudo sed -i 's|http://security.ubuntu.com|https://mirrors.aliyun.com|g' /etc/apt/sources.list
```

这一步会要求输入密码，输入密码时同样不会显示，输完之后直接回车就可以了

**更新软件包索引**

这一步是更新软件包索引

```plaintext
sudo apt update
```

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/DGt6bPhaio4hW6xXjbNcVuOKnEg.png)

### 3.2.安装所需基础环境

**git**：代码管理工具（下载、上传代码必备）

**curl**：网络请求工具（下载文件、访问接口必备）

**nodejs**：JavaScript 运行环境（前端 / Node 项目必备）

**npm**：Node 的包管理器（装 JS 库必备）

```plaintext
sudo apt install -y git curl nodejs npm
```

给npm更换为加速源

```plaintext
npm config set registry https://registry.npmmirror.com
```

## 四、hermes的安装和使用

### 4.1.Hermes Agent安装

https://hermes-agent.nousresearch.com/

```plaintext
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/FCakbHjkmolwEfxjUthcQTjYnEb.png)

这一步需要执行的时间非常长，执行完了之后会jin

### 4.2.配置大模型

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/UbvebB64WoduRqxGUg5cMetxngg.png)

**优先推荐minimax的token plan**，性价比高，选择minmax之后，然后进一步下一步输入apikey

> `beach_with_umbrella`
> **Token Plan订阅：**https://platform.minimaxi.com/subscribe/coding-plan?code=Jj0bqhwQT1&source=link

**比较推荐MiniMax的Token Plan**

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/Le0ebA8ovoexpBxmMNtcqYuHnSd.png)

**Hermes Agent +MiniMax 的搭配效果还是不错的~**

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/NbBybsKXCoGrjJxAor0c4uDnn3b.png)

输入apikey时依然不显示的，输入完之后直接回车就可以

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/GuAcbJnKnoS8LOxSnMTcVL5Hnab.png)

这一步直接回车就可以。

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/TSkXboTagotmehxE8W9c0qLXnqg.png)

模型选择MiniMax-M2.7，然后回车

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/Dcx7bgDL0oqo5hx45uYc04Dfntf.png)

这一步也是选择默认。

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/LUPPbUiIhowDwFxTZDic6pk5nAC.png)

这一步先选第二项，先测试一下hermes是否安装成功，然后再配置飞书。

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/GWsXbPG0NoGy5bxQG2DcIwNEnXg.png)

这一步直接输入Y

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/Ikwyb7VmXoHMaPxxpUMcCzzqnVy.png)

### **4.3.Hermes使用测试**

```plaintext
source ~/.bashrc
hermes
```

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/YIZXbtyNioC91Zx4VpAcxTxtnyd.png)

在这里可以问一句，如果能正常回复就成功了。

```plaintext
你好，请问你是什么大模型？
```

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/HwWJboWjVoWpBfx2zMecP0jKnVe.png)

### 4.4.配置飞书

```plaintext
hermes setup
```

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/TQAmbMMMFoJoYPxfgWqcmARSnSh.png)

还是Quick Setup，大模型的部分都跳过，一直到这一步

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/CYBDbqz4YoObNMxZr62c7DSJnLg.png)

选择第一个

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/PStpbeADRojrTKxAYDYcqZ8Bnhe.png)

选择飞书之后，进入下一步

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/AJrLbstUroSzajxMRy4c94kznjd.png)

选择第一项

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/ANl5bKtvYoYtx8xK2lAcuzZ6nZe.png)

然后拿出手机，打开飞书，扫描上面的二维码，成功之后，再回到命令行

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/YmUTbFLtXoJUBHxwaVAc1pywnje.png)

选择第一个

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/EFKJbKTLooZckMxc7yqc2dVXnFf.png)

选第一个

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/P9W6bHNP5oISU1xNaSTcviTYn8b.png)

输入y，这一步是自启动

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/H8hpb72hso1VeKxhVTIc17fun2d.png)

选第2个，设置网关为默认自启动

**飞书与hermes配对**

在飞书里跟你新生成的智能体发一条消息，

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/NsWAbzUUoo0H8BxZmqOcJZWJnMd.png)

然后再来到命令行里输入这段指令

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/AyJhbPY0OorXdOx8gKrcWdvJnjc.png)

然后回车之后提升成功就可以了

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/Vl21bWckCogTLJxbG5bcDUNFnSe.png)

再去飞书里发消息做一下测试，能正常回复就说明飞书已经搞好了。

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/Hts7bK13zootmzxu510cs2pBnNd.png)

## 五、Hermes使用体验提升

### 5.1.官方Dashboard

直接在hermes命令行中运行hermes dashboard

```plaintext
hermes dashboard
```

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/FgYLbBlwVoTJOvx3EGYcFXrgnOc.png)

看到出现这个就是成功了，其实装完Hermes之后，以后很多构建和安装的东西都可以让Hermes来完成。

接下来直接用

> `tada`
> **官方Dashboard：http://127.0.0.1:9119**

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/DGQ6bDyyHof51sxLplQcIq6Onec.png)

在这个界面就可以看到会话、定时任务、模型、Skill等基础配置，不过毕竟可惜的是会话部分只能看，无法直接进行chat。接下来我们就给他安装一个带这个功能的第三方webui。

### 5.2.第三方WebUI

比起官方的dashboard，这个是类似于openclaw可以直接聊天的，还可以打开多个会话。

> `gift`
> 仓库：https://github.com/nesquena/hermes-webui
> 最新版：0.50.96（2026-04-23）

#### **1.1 安装与运行**

```plaintext
帮我安装一下https://github.com/nesquena/hermes-webui
```

```plaintext
运行一下hermes-webui
```

#### 1.2 显示外部会话

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/W53LbPqAvoRj1gxTqhIcVTBynjc.png)

> `baguette_bread`
> **访问地址：**http://127.0.0.1:8787/

### **5.3.宝塔面板：构建一个“可视化”**

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/Ii4AbQPU1oQ85wxQmlocNoLnnHb.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/IlUqbFQDZof8mjxfoAsccaaunYd.png)

#### **3.1 安装宝塔面板指令**

```plaintext
wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh
```

#### **3.2 访问和使用宝塔面板**

安装完之后记好地址，以及用户名和密码，这里要注意将宝塔面板

```plaintext
https://127.0.0.1:31119/04b7e9f6（标黄的按你自己的来）
```

### 5.4.进化面板：让Hermes自进化可视化

#### 4.1 进化档案效果

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/RiinbXclPo8082xGQ2lc13BinOb.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/RqNBbXf12oGd7LxSM81cCXdCn9c.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/H6z4bgTlioXiuXxxlkycLd3mnkd.png)

#### **4.2 进化档案指令**

```plaintext
我有个想法，就是能把你的进化记录、进化的成果，专门做一个HTML来展示吗？而且以后可以每天都去更新一下这个HTML，这样我就能看到你的进化历程，到底进化在哪？比如：1. 进化的Skill 2. 包括记忆，就进化什么东西这些
```

#### 4.3 排版风格优化

如果生成的效果不好的话，可以用我的页面总结出来的风格去优化和调整一下。你也可以把风格描述给Hermes让他自己来调整

```plaintext
进化档案可以参照一下这个TXT文件，你去做一些调整和优化
https://cdn.hailuoai.com/mcp/cdn_upload/327407489771057158/388658179654272/1776829119_793b3324.txt
```

**txt里的内容如下：**

```xml
================================================================================
                    AI 进化可视化 HTML 页面制作指南
================================================================================

【用途】
制作一个"AI进化可视化"HTML页面，展示AI的Skills、记忆和进化历程。

================================================================================
                              页面结构
================================================================================

1. HEADER 头部
   - 头像: 120x120px圆形，带光晕动画，渐变边框
   - 标题: "XXX 进化记录" (渐变色)
   - 副标题: "AI 能力成长追踪"
   - 更新日期

2. STATS GRID 统计概览
   - 自适应4列网格布局
   - 每个卡片: 大数字 + 小标签
   - 示例: Skills数量、平台数量、能力数量等

3. EVOLUTION CARD 进化概览
   - 渐变背景卡片
   - 3列进化指标 (图标 + 数字 + 标签)

4. SKILLS SECTION 技能卡片
   - 自适应网格 (min-width: 280px)
   - 每个卡片包含:
     - 技能名称 + 分类标签
     - 描述文字
     - 技能标签 (小圆角标签)
     - 创建日期
   - 悬停效果: 上浮 + 顶部渐变线

5. MEMORY SECTION 记忆卡片
   - 分类标签 (用户/系统/核心)
   - 内容展示
   - 不同类型用不同颜色区分

6. TIMELINE 时间线
   - 左侧竖线 + 圆点
   - 每个节点: 日期 + 标题 + 描述卡片
   - 竖线渐变透明效果

7. FUTURE 未来方向
   - 同时间线结构
   - 展示规划中的进化方向

8. FOOTER 页脚
   - 作者信息
   - 底部说明

================================================================================
                              设计规范
================================================================================

【配色方案】
- 背景渐变: #0a0a0f → #1a1a2e → #16213e
- 强调色: #667eea → #764ba2 (紫蓝渐变)
- 文字色: #e0e0e0 (主) / #ccc (次) / #888 (弱)
- 卡片背景: rgba(255,255,255,0.03)
- 卡片边框: rgba(255,255,255,0.08)

【字体】
- 主字体: Noto Sans SC
- 备选: -apple-system, BlinkMacSystemFont, sans-serif

【间距】
- 卡片圆角: 16px
- 内边距: 24px
- 网格间距: 16-20px
- 区块间距: 60px

【交互效果】
- 卡片悬停: translateY(-4px) + 边框发光
- 头像动画: pulse (3s循环) 光晕变化
- 过渡: all 0.3s ease

【布局】
- 最大宽度: 1000px
- 自适应: grid + flexbox
- 响应式: 小屏幕2列，移动端1列

================================================================================
                              HTML 模板
================================================================================

<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>进化记录</title>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@300;400;500;700&display=swap" rel="stylesheet">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Noto Sans SC', sans-serif;
            background: linear-gradient(135deg, #0a0a0f 0%, #1a1a2e 50%, #16213e 100%);
            min-height: 100vh;
            color: #e0e0e0;
            line-height: 1.6;
        }

        /* 头部 */
        .header {
            text-align: center;
            padding: 60px 20px 40px;
        }
        .avatar {
            width: 120px; height: 120px;
            border-radius: 50%;
            margin: 0 auto 24px;
            /* 替换为实际头像URL或渐变背景 */
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            border: 3px solid rgba(102, 126, 234, 0.5);
            box-shadow: 0 0 60px rgba(102, 126, 234, 0.4);
            animation: pulse 3s ease-in-out infinite;
        }
        @keyframes pulse {
            0%, 100% { box-shadow: 0 0 60px rgba(102, 126, 234, 0.4); }
            50% { box-shadow: 0 0 80px rgba(102, 126, 234, 0.6); }
        }
        .header h1 {
            font-size: 2.5rem;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }
        .header .subtitle { color: #888; font-size: 1rem; margin-top: 8px; }

        /* 统计卡片 */
        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            gap: 16px;
            max-width: 900px;
            margin: 0 auto 50px;
            padding: 0 20px;
        }
        .stat-card {
            background: rgba(255, 255, 255, 0.03);
            border: 1px solid rgba(255, 255, 255, 0.08);
            border-radius: 16px;
            padding: 24px;
            text-align: center;
            backdrop-filter: blur(10px);
            transition: all 0.3s ease;
        }
        .stat-card:hover {
            transform: translateY(-4px);
            border-color: rgba(102, 126, 234, 0.5);
        }
        .stat-number {
            font-size: 2.5rem; font-weight: 700;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }
        .stat-label { font-size: 0.85rem; color: #888; margin-top: 4px; }

        /* 通用区块 */
        .section {
            max-width: 1000px;
            margin: 0 auto 60px;
            padding: 0 20px;
        }
        .section-title {
            display: flex;
            align-items: center;
            gap: 12px;
            font-size: 1.5rem;
            font-weight: 600;
            margin-bottom: 24px;
            padding-bottom: 12px;
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
        }

        /* 技能卡片网格 */
        .skills-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
            gap: 20px;
        }
        .skill-card {
            background: rgba(255, 255, 255, 0.03);
            border: 1px solid rgba(255, 255, 255, 0.08);
            border-radius: 16px;
            padding: 24px;
            transition: all 0.3s ease;
        }
        .skill-card:hover {
            transform: translateY(-4px);
            border-color: rgba(102, 126, 234, 0.3);
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
        }
        .skill-header { display: flex; justify-content: space-between; margin-bottom: 12px; }
        .skill-name { font-size: 1.1rem; font-weight: 600; color: #fff; }
        .skill-category {
            font-size: 0.7rem; padding: 4px 8px;
            background: rgba(102, 126, 234, 0.2); border-radius: 20px; color: #a5b4fc;
        }
        .skill-description { font-size: 0.9rem; color: #999; margin-bottom: 16px; }
        .skill-tags { display: flex; flex-wrap: wrap; gap: 6px; }
        .skill-tag {
            font-size: 0.75rem; padding: 4px 10px;
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 20px; color: #888;
        }

        /* 记忆卡片 */
        .memory-card {
            background: rgba(255, 255, 255, 0.03);
            border: 1px solid rgba(255, 255, 255, 0.08);
            border-radius: 16px;
            padding: 24px;
            margin-bottom: 16px;
        }
        .memory-type {
            display: inline-block; font-size: 0.75rem;
            padding: 4px 12px; border-radius: 20px; margin-bottom: 12px;
        }
        .memory-type.user { background: rgba(236, 72, 153, 0.2); color: #f472b6; }
        .memory-type.system { background: rgba(16, 185, 129, 0.2); color: #34d399; }

        /* 时间线 */
        .timeline { position: relative; padding-left: 30px; }
        .timeline::before {
            content: ''; position: absolute; left: 8px; top: 0; bottom: 0; width: 2px;
            background: linear-gradient(to bottom, #667eea, #764ba2, transparent);
        }
        .timeline-item { position: relative; padding-bottom: 40px; }
        .timeline-item::before {
            content: ''; position: absolute; left: -26px; top: 4px;
            width: 12px; height: 12px; border-radius: 50%;
            background: linear-gradient(135deg, #667eea, #764ba2);
            box-shadow: 0 0 20px rgba(102, 126, 234, 0.5);
        }
        .timeline-date { font-size: 0.8rem; color: #666; margin-bottom: 8px; }
        .timeline-content {
            background: rgba(255, 255, 255, 0.03);
            border: 1px solid rgba(255, 255, 255, 0.08);
            border-radius: 12px; padding: 20px;
        }
        .timeline-title { font-size: 1.1rem; font-weight: 600; color: #fff; margin-bottom: 8px; }
        .timeline-desc { font-size: 0.9rem; color: #999; }

        /* 页脚 */
        .footer {
            text-align: center; padding: 40px 20px;
            color: #666; font-size: 0.85rem;
        }
        .footer a { color: #667eea; text-decoration: none; }

        /* 响应式 */
        @media (max-width: 600px) {
            .header h1 { font-size: 1.8rem; }
            .stats-grid { grid-template-columns: repeat(2, 1fr); }
        }
    </style>
</head>
<body>
    <div class="header">
        <div class="avatar"></div>
        <h1>进化记录</h1>
        <p class="subtitle">AI 能力成长追踪</p>
    </div>

    <div class="stats-grid">
        <!-- 统计卡片 -->
    </div>

    <div class="section">
        <h2 class="section-title"><span>📈</span> 进化概览</h2>
        <!-- 进化概览内容 -->
    </div>

    <div class="section">
        <h2 class="section-title"><span>🛠️</span> 已掌握的 Skills</h2>
        <div class="skills-grid">
            <!-- 技能卡片 -->
        </div>
    </div>

    <div class="section">
        <h2 class="section-title"><span>🧠</span> 持久记忆</h2>
        <!-- 记忆卡片 -->
    </div>

    <div class="section">
        <h2 class="section-title"><span>📅</span> 进化时间线</h2>
        <div class="timeline">
            <!-- 时间线节点 -->
        </div>
    </div>

    <div class="footer">
        <p>Powered by AI</p>
    </div>
</body>
</html>

================================================================================
                              使用说明
================================================================================

1. 复制上述HTML模板
2. 替换以下内容:
   - 头像URL: 在 .avatar 样式中添加 background-image: url('头像链接')
   - 统计数据: 修改 .stat-number 和 .stat-label
   - Skills列表: 复制 .skill-card 模板，填入真实技能
   - 记忆内容: 复制 .memory-card 模板
   - 时间线: 复制 .timeline-item 模板

3. 保存为 index.html

4. 部署:
   - 本地: python -m http.server 8000
   - 或上传到服务器
   - 或使用 Mermini/MCP 等平台部署

================================================================================
                              颜色参考
================================================================================

主色调:
- #667eea (紫蓝起点)
- #764ba2 (紫蓝终点)
- #f093fb (粉色高光)

背景色:
- #0a0a0f (深黑)
- #1a1a2e (深蓝)
- #16213e (靛蓝)

文字色:
- #fff / #e0e0e0 (主文字)
- #ccc (次要)
- #888 (弱化)
- #666 (辅助)

================================================================================
                              文件信息
================================================================================

创建时间: 2026-04-22
原始文件: /workspace/evolution/index.html
部署地址: https://cplp4zw83x99.space.minimaxi.com

================================================================================
```

### 5.5 WSL Dashboard：WSL可视化操作

**SL Dashboard**

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/Z0Flbc6JUo4jVzxlKtHcUvA3nle.png)

> `pencil2`
> **下载地址：**https://github.com/owu/wsl-dashboard/releases

**如何导出镜像**

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/W39fboLWPobeJnxurAgc2SINnzd.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/QiMcbCz7goPGRDxTTLCcsSMTnMh.png)

**如何创建/还原镜像**

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/JrqzblG8FoY6gTxkMvicHNpmntc.png)

---

## 六、如果你感兴趣养虾养马真正用起来

**我是李君陌，专注于如何用AI来解决具体问题，**对这方面感兴趣的话可以加微信交流，**我的微信号：iamlijunmo**

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/SkVGb4OOPoBvbkxi4AjchBEinbe.png)

微信号

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/VaB8bssFYo6EBlxPfgacYumUnue.png)

公众号

**以下是我自己正在使用的几个小场景，更多案例会持续在知识星球更新 **

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/Hd6fbVLQpo5YnGxiAZYcLCF7nPb.png)

### 案例0：超级秘书的属性

***一个好的超级秘书一定是主动的，***

***当有跟你相关的重要事件会主动@你***

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/VN3abTWZ7oAAQtxJFDzcBsR2nLc.png)

***也会主动问要不要帮你盯着一些事情~***

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/GmxMbpnjro2qlWxUI1pcl2z9nXf.png)

### ***案例1：AI早资讯语音播报版***

每天早晨8点会给我推送AI资讯，并且转换成语音播报，可以在洗漱或者路上听。另外还有一个汇总页面，所有的早咨询都会保留下来。

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/XyeNbMzrSoZbU7xvy7gctY95nOf.png)

[view:2]

[语音播报.mp3](feishu-file:GamsbR0naoe4HJxUObqcrXxAnqh)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/Z7QTbDMmboB6U8xrvjmcgryjnSd.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/J0j4bRcDAoQQr5xdN6kcDWXsnfE.png)

### **案例2：同行与对标直播间监控**

做这个的目的是我打算也做一下抖音直播，所以要去关注一下同赛道的哪些博主做的比较好，什么样的形式，讲的什么内容，特别是这个在线直播人的数据，直接效果好的可以直接借鉴和参考，不好的就直接避坑了，也算是避开一个错误选择，节约试错成本。

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/LF1bb5ZVKo1Rh6x472tcREQAntg.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/XtBbbI6dyo5mO2xEIfscSS1FnQe.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/UGWmbViynoSNBVxaZiYcBqgznvc.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/P0xRbZGZuouaVlxCbx5cXdO2nLh.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/BmMabWcTQozhmBx74GscvSF1n2c.png)

这个是优化后的版本，增加了一些与我有关系的建议和分析（我跟agent说了我是做什么的这些详细信息了）

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/S5ALbASSHosfb2xq1N8c6il4nLc.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/TgjLbsGzaoSAA9xi1jYcYQlqnvg.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/NJLtbLPR9oHStBxpDErcSD9RnZR.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/WhZMbPyJGoe5C7xvRtDcWhw9nzf.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/Fyr7bd2bQoawcuxBGbrc6xr7nEe.png)

### 案例3：直接间挖掘用户真问题需求

这是专门盯着一个在线观看人数还比较多的

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/Zl37bSsmKoVF22xWnULcipZ5nzg.png)

### **案例4：低粉爆款找选题**

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/PF1TbBy0JozDyKxah3icYaConqc.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/FpSebQcZxokpr7xPIHlcFKrknhb.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/A2z2bLnXIor9vgxlZWRcF3Hjnp1.png)

### 案例5：评论区问题汇总挖选题（推送链接）

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/HoHxb517SoUpQbxDyDbcOri9n9K.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/CyRdbo2TOoxhOfx4OjNcJqBpn64.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/KFc2bktqHo2079x4D3IcnWz5nQb.png)

![](../../assets/2026-11-14-Hermes-OpenClaw实战进阶笔记/SlpObrWgho569ixU7mxcIXoZnKf.png)

### 案例6：招聘网站找B端客户

### 案例7：每天定时推送漫画（可定制）单词卡

### 案例8：电商同行产品负评挖掘分析

### 案例9：招聘网站挖掘成长行业与项目

### 案例10：自动检索对标博主新内容的知识点以PPT推送

### 案例11：定时对标直播间话术改写（完成录制+文案提取+自动改写）
