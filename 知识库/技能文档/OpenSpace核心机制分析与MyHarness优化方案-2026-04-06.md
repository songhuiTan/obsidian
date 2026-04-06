# OpenSpace 核心机制分析与 MyHarness 优化方案

> 归档时间：2026-04-06
> 基于 OpenSpace (HKUDS) 源码研究

---

## 一、OpenSpace 的核心创新

OpenSpace 不是一个 harness，而是一个**自进化 Skill 引擎**。它的核心命题：

> Skill 不应该是静态的 Markdown 文件，而应该在每次使用中学习、适应、进化。

### 1.1 三种进化模式

```
FIX（修复）     → 技能没起作用？分析原因，就地修复
DERIVED（派生） → 从已有技能组合出新的更强技能
CAPTURED（捕获）→ 发现了新模式？创建全新的技能
```

### 1.2 三个进化触发器

```
触发器 1：执行后分析（每次执行后自动分析技能是否有效）
触发器 2：工具退化检测（某工具成功率下降时触发）
触发器 3：周期性指标检查（每 5 次执行检查一次整体质量）
```

### 1.3 技能谱系（Lineage）

```
skill_v1 (IMPORTED, gen=0)
  └── skill_v2 (FIXED, gen=1) — "修复了 curl 参数格式"
       └── skill_v3 (FIXED, gen=2) — "增加了错误处理"

skill_A (IMPORTED, gen=0)
skill_B (IMPORTED, gen=0)
  └── skill_C (DERIVED, gen=1) — "组合了 A 和 B 的能力"
```

每个技能版本都有：
- `parent_skill_ids`（父母是谁）
- `generation`（第几代）
- `change_summary`（改了什么）
- `content_diff`（具体差异）
- `content_snapshot`（完整快照）

### 1.4 技能质量指标

```python
applied_rate     = total_applied / total_selections    # 被选中后实际使用率
completion_rate  = total_completions / total_applied   # 使用后任务完成率
effective_rate   = total_completions / total_selections # 端到端有效率
fallback_rate    = total_fallbacks / total_selections   # 回退率（技能不可用）
```

### 1.5 混合搜索（BM25 + Embedding）

```
候选技能 > 10 个？
├── 先用 BM25 粗排
├── 再用 Embedding 精排
└── 最后用 LLM 选出最合适的
```

---

## 二、MyHarness 当前缺失 vs OpenSpace 能力

| 维度 | MyHarness 现状 | OpenSpace 能力 | 优化方向 |
|------|--------------|---------------|---------|
| **技能进化** | 静态 SKILL.md，永不改变 | FIX/DERIVED/CAPTURED 自动进化 | 加入 Skill Evolution Skill |
| **执行分析** | 无 | 每次执行后分析技能是否有效 | 在 Sprint COMPOUND 阶段加入执行分析 |
| **质量指标** | 无 | applied_rate/completion_rate 等 | 在 docs/progress/ 中跟踪质量 |
| **技能谱系** | 无 | 父子关系、版本链、diff | 加入 .skill-lineage 侧文件 |
| **技能安全** | 无 | 正则扫描恶意模式 | 加入安全检查 skill |
| **模糊匹配** | 无 | 6 级降级模糊匹配 | 可用于 skill patching |

---

## 三、优化方案：6 个增强

### 增强 1：Skill Evolution Skill（自进化）
新增 skill 文件，在 Sprint 结束后不仅 compound 知识，还进化 skill 本身。

### 增强 2：执行分析协议
在 session-bridge.md 中加入 skill 使用评估，跟踪哪些 skill 有效、哪些需要修复。

### 增强 3：技能质量跟踪
在 docs/progress/ 中新增 skill-metrics.md，记录每个 skill 的使用统计。

### 增强 4：技能谱系
为每个 skill 添加 .lineage.json 侧文件，跟踪版本演进。

### 增强 5：安全检查 Skill
新增 safety-check skill，在执行任何非只读 skill 前检查安全性。

### 增强 6：CLAUDE.md 增强
在引擎指令中加入 skill 进化触发条件和分析协议。

---

## 四、不纳入的部分（及原因）

| OpenSpace 特性 | 不纳入原因 |
|---------------|-----------|
| Python 后端引擎 | MyHarness 是 Claude Code 原生，不需要 Python |
| GUI 自动化 | Harness 不需要桌面操作 |
| 云端技能分享 | 过于复杂，后续考虑 |
| 嵌入向量搜索 | Claude Code 没有原生 embedding 支持 |
| SQLite 存储 | 用 Markdown 文件更简单 |
| GDPVal 基准测试 | 属于 OpenSpace 的评估框架 |

---

## 标签

#ClaudeCode #OpenSpace #SkillEvolution #自进化 #MyHarness
