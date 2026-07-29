# Archify Skill：让 Agent 直接出架构图

> **来源**：微信公众号「阿飞AI实操日记」(阿飞)
> **日期**：2026-07-11
> **链接**：https://mp.weixin.qq.com/s/Mx7GN07Sos67xeN2dPcyRA
> **标签**：`Archify` `架构图` `Agent Skill` `Claude Code` `Codex CLI` `HTML` `

---

## 简介

Archify 是一个 Agent Skill（非普通画图软件），装进 Claude Code、Codex CLI、opencode 等 Agent 工具后，直接用自然语言生成技术图。

GitHub：https://github.com/tt-a1i/archify

---

## 支持的图类型

| 类型 | 适合场景 |
|------|---------|
| Architecture diagram | 系统架构、服务依赖、模块关系 |
| Workflow diagram | 审批流、发布流、业务流程 |
| Sequence diagram | 请求链路、服务调用顺序 |
| Data-flow diagram | 数据流转、ETL、日志链路 |
| Lifecycle diagram | 订单状态机、任务生命周期 |

---

## 核心特性

- **对话式出图**：描述系统关系即可，无需手动拖框
- **自然语言驱动**：甚至可直接让 Agent 分析代码仓库后输出架构图
- **产物为自包含 HTML**：发给同事直接打开，无需额外依赖
- **内置深浅两套主题**：适合不同文档场景
- **导出格式**：PNG / JPEG / SVG / WebP，支持最高 4 倍分辨率
- **SVG 支持自动适配系统主题**：深色模式看深色图、浅色模式看浅色图

---

## 使用方式

最省事的安装：

```bash
# 直接在 Agent 对话中说：
帮我安装一下这个 Skill，https://github.com/tt-a1i/archify
```

首次使用时若报依赖缺失，继续让 Agent 自动补装即可。

---

## 适合场景

1. **写技术方案**：文字写了半天读者还是不明白服务怎么流动，补一张架构图
2. **项目交接**：让 Agent 扫代码库整理出模块关系图
3. **写 README 或技术文章**：流程图让读者更快判断工具解决什么问题

---

## 注意

- 自动布局、连线避让、中文文本宽度估算还有优化空间
- 建议：先让 Archify 出初稿，再在对话中继续调整
- 复杂图需检查箭头方向、节点分组、中文标签换行

> 工具负责生成图，我们负责判断这张图有没有把问题讲明白。
