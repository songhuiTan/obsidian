---
source: 微信公众号
author: 阿飞AI实操日记（阿飞）
url: https://mp.weixin.qq.com/s/MTpEcA0NCo8F-ZHMazPK9g
date: 2026-06-24
tags: [Ubuntu, Linux, Windows, 系统安装, Rufus, Ventoy, Hermes Agent]
---

# 把一台 Windows 电脑，彻底变成 Ubuntu Linux 是什么体验？

> 背景：作者在 Windows 上用 WSL 跑 Hermes Agent，但随着 Agent 越来越复杂，遇到各种边界问题（性能、浏览器自动化、网络环境），决定把闲置 Windows 电脑直接改成原生 Linux。

## 需要准备什么

1. **一个 ≥10GB U 盘**（别用杂牌）
2. **Ubuntu 22.04 Desktop ISO**（选 Desktop 版才有图形界面）
   - 下载：https://releases.ubuntu.com/22.04/
   - 选 `ubuntu-22.04.5-desktop-amd64.iso`
3. **启动工具**：推荐 Rufus（比 Ventoy 更稳）

## 踩坑记录

作者先用 Ventoy，反复报错，换成 Rufus 一次成功。

## 操作步骤

### 1. 制作启动 U 盘

1. 下载 Rufus：https://rufus.ie/zh/
2. 双击 `rufus.exe`
3. 推荐设置：
   - **设备**：你的 U 盘
   - **启动类型**：选择 Ubuntu ISO
   - **分区类型**：GPT
   - **目标系统**：UEFI
   - **文件系统**：FAT32
4. 点击开始

> 不同电脑配置不同，把截图发给 GPT 帮你选参数。

### 2. 进入 BIOS 启动 U 盘

1. 进 BIOS 修改 GSM Support 等设置（不懂的问 GPT）
2. 插 U 盘，重启，按 F12 / F11 / DEL（视电脑而定）
3. 选择 **Try or Install Ubuntu**
4. 选 **Install Ubuntu**
5. 按提示 Next 走完安装流程

## 完成后体验

- 有图形界面，使用与 Windows 没什么两样
- 大部分软件支持 Linux
- 可通过 SSH 或 VNC 远程操作
- 原生 Linux 环境跑 Agent 比 WSL 舒服很多
