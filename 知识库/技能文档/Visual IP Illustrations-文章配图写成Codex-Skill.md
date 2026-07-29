# Visual IP Illustrations：文章配图这事，居然被写成了一个 Skill

> 来源：Github开源项目（微信公众号）  
> 归档：2026-07-05  
> 链接：https://mp.weixin.qq.com/s/E_2h-rvhIb6HIyQRS7V_XA  
> 标签：#AIGC #内容创作 #Codex #Skill #配图

---

## 文章概述

一个将"文章配图"拆解为 Codex Skill 的开源项目。不是简单丢一句 prompt 让 AI 画图，而是先读文章核心判断 → 选视觉 IP 路线 → 生成一组 16:9 手绘风正文插图。

**GitHub 仓库**：yangchuansheng/visual-ip-illustrations

---

## 核心设计

### 多角色视觉 IP 路线
每个角色有独立的路由、别名、输出目录、风格规范，附带 QA gate 和边界说明：

| 角色 | 适用场景 |
|------|---------|
| 小黑（默认） | 通用科技/产品插图 |
| 纸盒人 | 轻量/趣味内容 |
| Rust 螃蟹 Ferris | Rust 工程化内容 |
| Go Gopher | Go 后端内容 |
| 连帽衫海豹 | 轻松话题 |
| OpenClaw | OpenClaw 生态 |
| 蔡徐坤（gated-public-figure） | ⚠️ 有授权/商标限制，需注意边界 |

### 关键价值
- **风格统一**：连续阅读时角色/画风一致，避免公众号配图"第一张手绘、第二张 3D、第三人比例飘"的问题
- **按内容匹配角色**：AI Agent 文章让小黑搬流程图，Rust 文章让 Ferris 抱积木修系统，Go 后端让 Gopher 拎管道跑
- **批量产出**：4~8 张同风格插图，适用于科技号、产品笔记、方法论文章

### 注意事项
- 公开人物、社区吉祥物等路线有明确的授权、商标、代言边界
- 目标不是"多精致"，而是"让整篇文章看起来像同一个视觉系统里出来的"
- 最适合公众号连载/系列文章的配图一致性需求

---

## 与我当前工具的关联

当前 Hermes 生态中已有 `image_generate` 工具和 `baoyu-comic`/`baoyu-infographic` 等创作型 Skill。此项目的**角色路由+QA gate** 思路可借鉴到当前的内容配图工作流中，为每类文章预置一个"视觉 IP"，从 prompt 随机发挥变为风格约束下的批量产出。
