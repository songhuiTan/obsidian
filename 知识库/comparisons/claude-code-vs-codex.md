---
title: "Claude Code vs Codex — 两大 AI 编程终端 Agent 全面对比"
created: 2026-07-21
updated: 2026-07-21
type: comparison
tags: [comparison, claude-code, codex, tool, harness]
sources:
  - 技能文档/2026-06-15-Codex桌面端教程-30分钟从入门到精通零基础保姆级教程.md
  - 技能文档/Claude Code官方Loop Engineering教程.md
  - 技能文档/2026-06-10-从 Claude Code 动态工作流看 Agent Harness 设计.md
  - 技能文档/2026-06-08-人类对Codex的开发不足1%：干货长文汇总Codex最新玩法、技巧和解答.md
  - 技能文档/2026-07-02-CoreCoder-Claude-Code底层开源复刻.md
  - 技能文档/2026-06-22-Claude-Code-loop循环-自动化开发工作流.md
  - 技能文档/2026-06-04-CodeX-5-skills-指北师.md
confidence: medium
---

# Claude Code vs Codex

## 两大 AI 编程终端 Agent 全面对比

| 对比维度 | Claude Code | Codex (CodeX) |
|----------|-------------|---------------|
| 开发商 | Anthropic | 开源社区 |
| 核心模型 | Claude 系列模型 | 多模型支持（社区可切换） |
| 交互方式 | Terminal CLI | Terminal CLI + Prompt Canvas |
| 工作流模式 | Plan-Execute-Verify 循环 | Skill 驱动 + 无限画布 |
| Skill 体系 | Skill 系统，可扩展 | 5 Skills 指北师体系，更自由 |
| 开源状态 | 闭源，有社区复刻（CoreCoder） | 开源生态 |
| 桌面端支持 | 官方 Terminal | 桌面端教程 + Prompt Canvas |
| 文档处理 | 一般 | Prompt Canvas 支持图片迭代 |
| 社区生态 | 较大，Anthropic 官方支持 | 活跃，但更分散 |
| 衍生项目 | CoreCoder, free-code, ai-job-search | Fable5 工作流继承, 原型 Skill |
| 企业支持 | Anthropic 商业支持 | 社区支持 |
| Loop 工程 | 官方 Loop 教程完善 | 进阶教程（模型变强后重做流程） |

## 架构差异

**Claude Code** 采用动态工作流设计，核心是 Loop Engineering 的 Plan-Execute-Verify 循环。它通过与 Claude 模型的深度集成，实现代码理解、生成、编辑和调试。底层有 Skill 系统支持扩展，已衍生出 CoreCoder（开源复刻）、ai-job-search 等生态项目。

**Codex** 则更强调 Skill 生态的开放性和自由度。"人类对 Codex 的开发不足 1%"的评价充分说明其扩展潜力。其 5 Skills 指北师体系提供了灵活的能力扩展路径。Prompt Canvas 是一个差异化亮点——支持无限画布和图片迭代，这在纯 Terminal 工具中较为罕见。

## 生态与社区

- **Claude Code** 背后有 Anthropic 的商业支撑，生态相对集中。CoreCoder 项目提供了底层复刻，free-code 提供免费编译版。社区围绕 Claude Code 开发了丰富的 Skill 和 Harness 集成方案。
- **Codex** 生态更为开放和分散。Fable5 下架后其工作流能力被 Codex 继承，体现了平台整合能力。免费开源原型 Skill（烧了 1 亿 Token）项目展示了社区对 Codex 的热情。

## 选型建议

**选择 Claude Code 当：**
- 需要稳定的官方支持和商业保障
- 偏好 Claude 模型的代码生成质量
- 需要成熟的 Loop 工程方法论指导
- 作为 [[langgraph]] 或 [[deep-agents]] 的编程执行引擎

**选择 Codex 当：**
- 偏好开源生态和自由度
- 需要 Prompt Canvas 的可视化交互
- 想要更灵活的 Skill 扩展体系
- 需要多模型切换能力或图片迭代工作流

## 未来趋势

两者都在快速演进。Claude Code 正被越来越多的 Harness 框架作为底层执行引擎集成，而 Codex 的开放生态使其在技能多样性和创新速度上具有优势。Claude Code 和 [[claude-code]] 的对比已从"谁更好"转向"针对什么场景选什么工具"——企业环境更适合 Claude Code，个人开发者/社区场景更适合 Codex。
