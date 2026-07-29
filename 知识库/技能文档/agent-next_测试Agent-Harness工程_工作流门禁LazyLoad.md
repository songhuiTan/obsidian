---
tags:
  - Agent
  - Harness工程
  - 测试
  - CI/CD
  - 工作流
source: 微信公众号 CyberCosmos
author: Phoenine
date: 2026-07-12
status: completed
---

# agent-next：把测试 Agent 接进流水线——工作流、门禁与 Lazy Load 的 Harness 实践

> 公众号 CyberCosmos · Phoenine · 2026年7月9日

## 核心矛盾

**测试 Agent 能生成，但不敢直接用。** 差距不在模型能力，在**过程能否被检查、复现和阻断副作用。**

| 阶段 | 测试 Agent 的对应表现 |
|------|----------------------|
| 能生成 | 能写用例、能写报告、能复述需求 |
| 不敢上线 | 产物路径乱、阶段乱跳、副作用不可控、无法复现 |
| 中间差什么 | **Harness：路由、阶段、证据、门禁、确认** |

**agent-next** — 以 Git 仓库为载体的测试 Agent 操作系统。不是「更大的 Prompt」，是可版本化、可机器校验、可跨 Cursor/Hermes 复用的仓库结构。

---

## 五层 Harness 架构

### ① 入口层：第一屏加载

| 组件 | 物理位置 | 作用 |
|------|---------|------|
| 硬约束 | AGENTS.md | Cursor 等工作区自动注入 |
| 会话硬约束 | Hermes SOUL.md | 每个新会话注入 |
| Router Skill | skills/agent-next/SKILL.md (~70行) | 选 entry、声明 phase、指向 phase_doc |
| 路由索引 | workflows/index.md | 三入口 + Skill 索引 + 首次动作清单 |

Router **极薄**——不做写用例、上传缺陷、跑自动化，只做调度和停损（gate 失败即停）。

### ② 流程层：阶段即文件

```bash
python3 tools/phase_doc.py --entry bug-regression --phase "Change Scope"
# → workflows/bug-regression/phases/02-change-scope.md
```

阶段名的唯一权威在 `tools/phases.py`，与 `stage_gate.py` 共用——文档与代码不一致时，以代码为准。

### ③ 技能层：窄 Skill + references

| 职能 Skill | 负责 | Bug 回归典型阶段 |
|-----------|------|-----------------|
| 需求/缺陷摄入 | 读 PRD、Bug 单 | Bug Intake |
| 测试点/影响分析 | 变更范围、风险 | Change Scope, Impact |
| 测试用例 | 用例设计 | Coverage Match |
| 自动化 | 分类、执行 | Decision 后可选 |
| 报告 | 结项追溯 | Regression Report |

格式细则在 `skills/*/references/` 中——例如用例标题必须是 `TC-001：业务描述`，禁止 `VC-01`。

### ④ 证据层：三类数据

- **运行态** `runs/` — 每个任务的 state.json
- **交付物** `outputs/` — 产物 v1/v2
- **领域知识** `knowledge/` — 可复用，与来源证据分离

state.json 核心字段：`entry/phase`（位置）、`workflow`（审计路径）、`skill_receipts[]`（读过什么+sha256）、`repository_evidence[]`（看过哪些文件）、`artifacts[]`（产物记录）、`confirmations[]`（副作用确认）。

### ⑤ 机器层：脚本做门禁

| 工具 | 作用 |
|------|------|
| `run_state.py` | 创建/更新 state.json |
| `phase_doc.py` | 解析当前阶段路径 |
| `copy_template.py` | 从 templates/ 生成带 frontmatter 的产物 |
| `validate_test_cases.py` | 标题、章节、receipt 机审 |
| `stage_gate.py` | 当前 phase 能否标记完成 |

---

## 三条 Workflow

| Entry | 何时用 | 阶段数 |
|-------|--------|--------|
| `feature-testing` | PRD、需求单、特性 MR | 8（含 Optional） |
| `bug-regression` | Bug ID、修复 MR | 8 |
| `release-acceptance` | 版本、tag、发布验收 | 6 |

### Bug 回归八阶段

1. **Bug Intake** — 锁定缺陷上下文与修复来源（只有 notes，无 artifact 文件）
2. **Change Scope** — 弄清改了什么（change_scope 模板文件）
3. **Impact Analysis** — 影响面与风险
4. **Coverage Match** — 现有覆盖与缺口
5. **Decision Gate** — 选执行路径
6. **Regression Plan** — 策略与补用例
7. **Execution** — 执行（可选）
8. **Regression Report** — 结项

---

## Lazy Load 改造：从 400+ 行到 ~100 行

### 旧模式问题

| 载体 | 旧形态 | 问题 |
|------|--------|------|
| Router SKILL.md | ~160行，混有路由/阶段说明/踩坑 | Router 变第二本 workflow |
| Workflow | 单体 ~370行 | 八阶段全文一次读入，后半段规则被截断 |
| stage-gates.md | ~630行散文+表格 | Agent 重复通读 |
| 下游 Skill | 指向单体 workflow | 路径过期且不感知 |

### 改造后

| 职责 | 位置 | 行数 |
|------|------|------|
| 指路 | Router + index.md | Router ~70行；index ~83行 |
| 路由表 | workflow/README.md | ~55-65行 |
| 阶段动作 | phases/NN-*.md | ~15-35行/phase |
| 门禁 | stage-gates 机器表 + stage_gate.py | ~174行 |

**改造四步：**
1. 拆 workflow：README + phases（每个阶段一个独立文件）
2. 加解析器：phase_doc.py + phases.py（避免「Change Scope」vs「Scope Change」漂移）
3. 瘦 Router，重 Pitfalls：事故教训 offline 合并，不进 phase 正文
4. 压缩 stage-gates：保留机器表，删除逐阶段散文复述

**量化收益：** 八阶段规则类 Token 从 400+ 行→~100 行，整体约省一半 Token。

---

## 案例对照：Bug #1649

### 无 Harness
```
用户给 Bug ID + "浏览器验证"
→ 直接写 VC-01 用例
→ 猜 API 端口/路径（与实际网关路径不符）
→ 产物写在 runs/
→ 无合规 state.json
```

### 有 Harness（阶段不可跳）
```
1. skill_view("agent-next")→index→bug-regression
2. phase_doc → Bug Intake; notes: bug_surface: backend
3. stage_gate PASSED → Change Scope → 模板产物
4. Impact → Coverage → decision_path: supplement_cases
5. copy_template → validate → stage_gate
6. Execution：沿用用户 Network 面板里的 URL 与 Cookie
7. regression_report + traceability
```

---

## Cursor 与 Hermes 双端落地

| 载体 | 作用 | 注意 |
|------|------|------|
| 仓库 AGENTS.md | Cursor 自动注入 | — |
| Hermes SOUL.md | 每会话注入 | 须手写 agent-next 硬规则 |
| skills/ rsync 到 profile | Hermes Skill 对齐 | `rsync -a --delete skills/ ~/.hermes/profiles/coding/skills/` |
| workflows/ 留在项目仓库 | 须在根执行 | phase_doc/stage_gate 依赖仓库 |

**踩坑：** `~/.hermes/profiles/coding/AGENTS.md` 不会自动注入，规则写在 SOUL.md 或通过 `skill_view("agent-next")` 加载。

---

## Skill 自进化

企业场景中，单轮轨迹直接写回 Skill 容易过拟合。推荐路径：

> 多轮踩坑 → **离线归纳** → **人审** → 合并进 Skill/tools → rsync 到 Hermes profile

| 沉淀位置 | 适合什么 |
|---------|---------|
| 会话 MEMORY | 个人习惯、本机路径 |
| skills/ + workflows/ | 团队一致规则 |
| tools/stage_gate.py | 必须 100% 执行的校验 |

---

## 一句话建议

若已在用「一个大 Skill + 长 workflow」，不要先加 Prompt，先问四个问题：
- 阶段能否一文件一阶段？
- 路径能否机器解析？
- Router 能否瘦到只指路 + Pitfalls？
- Gate 能否表格 + 脚本？

四问都能答「是」，再改下游 Skill 并做一次**故意删除旧 stub 的破坏性发布**——逼 Agent 和新会话走新路径。

> Lazy Load 的价值不在少写几百行 Markdown，而在让「当前阶段该读什么」变成可执行、可审计的单次动作。

---

## 参考

- 微信公众号：CyberCosmos 原创
- 作者：Phoenine
- 原文：https://mp.weixin.qq.com/s/up4IzyXimoVn_5D4n1RaiQ
