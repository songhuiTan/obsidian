# 一个改变我使用AI 编程 workflow 的开源工具

> **来源**：KEEN的创享 公众号  
> **日期**：2026-06-29  
> **原文链接**：https://mp.weixin.qq.com/s/pK8wuIFkmxUsrf4GCWUS1g  
> **分类**：Claude Code & Codex

---

## 文章概要

介绍 NeoLabHQ 的 **Context Engineering Kit (CEK)** — 一套专为 Claude Code、Cursor、OpenCode 等 AI 编码代理设计的上下文工程工具包，将 AI 编码从"碰运气"推向"可工程化"。强调极致 Token 高效、按需激活插件，有真实生产数据背书。

---

## 核心插件

### Reflexion — 自动纠错器

- 基于 Self-Refine 和 Reflexion 论文
- 写完代码后触发 `/reflect`，Claude 自反思、找问题、提改进建议
- 配合 `/memorize` 把教训固化到 CLAUDE.md，越用越聪明

### SDD（Spec-Driven Development）— 生产级神器

- 完整流程：setup → specify → plan → implement → document
- 集成 architect、reviewer 等子代理
- 深度集成 DDD、SOLID、Clean Architecture 等最佳实践
- Muda 浪费分析，减少代码复杂度
- v2.0 后生成可用代码成功率 **高达 99%**

### SADD（Subagent-Driven Development）— 轻量并行利器

- 支持 do-and-judge、do-in-steps 等模式
- 每个子任务用新鲜上下文，避免大上下文污染
- 质量门 + 代码审查，迭代快且稳

### Review + Git — 代码审查与流程闭环

- Review：多专业代理（bug-hunter、安全审计、测试覆盖）审代码和 PR
- Git：规范 commit、创建 PR、处理 review comments

### 其他插件

- **DDD**：注入架构原则
- **Kaizen**：根因分析
- **Tech Stack**：语言最佳实践自动注入

---

## 可靠性对比数据

| 方法 | 小任务 | 大任务（20+ 文件变更） |
|------|--------|----------------------|
| 纯 one-shot 提示 | 60-80% | 1-20% |
| Reflexion (/reflect + /memorize) | 显著提升 | 避免重复犯错 |
| SADD / SDD | — | 70-95%（配合人工 review 极稳） |

---

## 安装方式

```bash
# Claude Code
/plugin marketplace add NeoLabHQ/context-engineering-kit

# Cursor / Codex 等
npx skills add NeoLabHQ/context-engineering-kit
```

**文档**：https://neolab.gitbook.io/cek  
**仓库**：https://github.com/NeoLabHQ/context-engineering-kit

---

## 关键词

`Context Engineering Kit` `CEK` `NeoLabHQ` `SDD` `SADD` `Reflexion` `Spec-Driven Development` `Subagent-Driven Development` `Claude Code` `Cursor` `OpenCode` `上下文工程` `AI编码工作流`
