---
source: 微信公众号
author: 极客之家（小黑）
url: https://mp.weixin.qq.com/s/hg91AEbxHbRiRl86-hLquA
date: 2026-06-23
tags: [harness-anything, Agent, CLI, WPS, Office, Photoshop, Illustrator, Zotero, COM, Windows]
---

# GitHub 又一神器！47 个命令让 AI Agent 直接操控 WPS、PS 和 Illustrator

harness-anything — 让 AI Agent 通过 CLI 直接操控桌面软件的桥接层。

开源地址：https://github.com/yb2460/harness-anything

## 核心思路

把鼠标点点点的操作全部变成命令行。AI Agent 不用理解 GUI，发命令就行。底层走 Windows COM 自动化接口，WPS/MS Office 和 Adobe 全家桶原生支持。

**缺点：Windows only**（COM 是 Windows 专属），需正版 Adobe / WPS / MS Office。

## 五大模块

### 1. cli-anything-wps：Office 变提线木偶

封装 **47 个 CLI 命令**，走 COM 操控 WPS / MS Office：

- **Writer**：插段落、设标题、画列表、建表格、塞图片、查找替换、改字体
- **Calc**：增删工作表、读写单元格、输公式、批量填充、合并单元格
- **Impress**：新建幻灯片、删页、改文本框、画形状、换背景、导出

导出格式：DOCX、XLSX、PPTX、PDF、TXT、HTML、CSV、RTF。PPT 自带 4 套主题 + 14 种布局 + 5 维度质量审查。

```bash
pip install git+https://github.com/yb2460/cli-anything-wps.git
cli-anything-wps document new --type impress --name "演示"
cli-anything-wps preset apply academic --talk-type defense
cli-anything-wps export render output.pptx -p pptx
```

### 2. cli-anything-zotero：学术流水线

Zotero 文献管理 + **27 个学术 Skill**，覆盖完整学术工作流：

- **search**：文献检索
- **research**：创意与假设
- **writing**：论文撰写
- **review**：审稿
- **visualization**：图表
- **analysis**：统计
- **pipeline**：完整学术流程

```bash
cli-anything-zotero skills list
cli-anything-zotero skills pipeline original_article
cli-anything-zotero skills journal "Nature"
```

### 3. illustrator-harness：矢量遥控

Adobe Illustrator COM 桥接（`Illustrator.Application`）：

- 新建/打开/保存 AI 文档
- 增删改图层、调可见性、上锁
- 画矩形、椭圆、线条、多边形
- 加文字、改字体、调大小、换颜色
- 导出 PNG、JPEG、SVG、PDF

```bash
cli-anything-illustrator project new logo.ai -w 500 -h 500
cli-anything-illustrator text add "Brand" --x 100 --y 100 --font "Arial" --size 72
cli-anything-illustrator export svg output.svg
```

### 4. photoshop-harness：位图遥控

Adobe Photoshop COM 桥接：

- 新建/打开/保存 PSD，调尺寸/分辨率/色彩模式
- 图层操作：增删改、可见性、透明度、混合模式
- 选区操作：全选、羽化、反选、扩展
- 裁切、旋转、翻转、改画布大小
- 文字图层、滤镜
- 导出 PNG、JPEG、WebP、PSD

```bash
cli-anything-photoshop project new poster.psd -w 1920 -h 1080
cli-anything-photoshop text add --content "Hello World" --font "Arial" --size 72
cli-anything-photoshop export png --output result.png
```

### 5. WPS PPT 自动化（演示案例）

JSON 数据 + 元素路由 + WPS COM → 全自动 PPT 生成。案例涵盖五所高校招生数据 PPT（每校 9-14 页，各校主题色，13 种元素类型）。

## 系统要求

- Windows 10/11
- WPS Office 2019+ 或 Microsoft Office 2016+
- Zotero 7+（可选）
- Adobe Illustrator 2023+（可选）
- Adobe Photoshop 2023+（可选）
- Python 3.10+，pywin32

## 关键信息

- **开源协议**：MIT
- **GitHub**：github.com/yb2460/harness-anything
- **注意**：COM 接口与 MS Office VBA 兼容，切到 MS Office 只需改 ProgID
