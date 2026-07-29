# Comet：基于 OpenSpec + Superpowers 双星驱动的 SpecSkill

> 来源：公众号「职场向上生长力」（作者：文/职场向上）
> 时间：2026-06-12
> 归档：2026-07-02
> GitHub：rpamis/comet

## 一、定位

Comet 是架设在 **OpenSpec（WHAT）** 和 **Superpowers（HOW）** 之间的编排层，负责 **WHEN & NEXT**。

- **OpenSpec** → 提案管理、规格生命周期、delta spec 同步、归档
- **Superpowers** → 头脑风暴、设计文档、计划拆分、TDD、子代理执行、代码审查
- **Comet** → 阶段状态管理、自动化过渡、校验、归档衔接

> "OpenSpec 管理了 WHAT 但 HOW 不够细，Superpowers 管理了 HOW 但缺少完整的生命周期 closure。Comet 把两者串联起来。"

## 二、五阶段工作流

```
/comet-open  →  /comet-design  →  /comet-build  →  /comet-verify  →  /comet-archive
(OpenSpec)      (Superpowers)     (Superpowers)     (Both)            (OpenSpec)
```

### 阶段 1：Open（OpenSpec propose）
- 创建 `openspec/changes/<name>/` 目录
- 生成 proposal.md、design.md、tasks.md、specs/
- 等你审核确认才进入下一步

### 阶段 2：Design（Superpowers brainstorming）
- 苏格拉底式追问，2-3 个方案带推荐
- 自动生成设计决策文档到 `docs/superpowers/specs/`

### 阶段 3：Build（Superpowers 子代理开发）
- writing-plans → tasks.md 细化到原子 step
- subagent-driven-development（Implementer → Spec Reviewer → Quality Reviewer）
- **隔离模式**：强制 branch 或 worktree
- **构建模式锁定**：Guard 防止跳过构建决策

### 阶段 4：Verify（强制检查）
- 测试全部通过
- tasks.md 全部勾选
- 实现符合 design doc 目标
- **必须 `verify_result=pass` 且 `branch_status=handled` 才可进入下一阶段**

### 阶段 5：Archive（OpenSpec 归档）
- delta spec → 主 spec 同步
- 标记文档状态，移动 archive 目录

### 快捷模式
| 模式 | 命令 | 跳过 |
|------|------|------|
| Hotfix | `/comet-hotfix` | Brainstorming |
| Tweak | `/comet-tweak` | Brainstorming + Full Plan |

## 三、状态机：`.comet.yaml`

每个 change 目录下的 YAML 护照，是 AI Agent 的"外部记忆"：

```yaml
workflow: full
auto_transition: true
phase: build
build_mode: subagent-driven-development
isolation: branch
design_doc: docs/superpowers/specs/...-design.md
plan: docs/superpowers/plans/...md
verify_result: pending
branch_status: pending
archived: false
handoff_hash: sha256:a1b2c3...
```

- **阶段定位** — AI 读 `phase` 字段即知当前阶段
- **交接校验** — `handoff_hash` SHA256 指纹确保交付物未被篡改
- **中断恢复** — 换终端、重启 Claude Code 状态都在

## 四、脚本体系

| 脚本 | 职责 |
|------|------|
| `comet-guard.sh` | 阶段过渡守卫 |
| `comet-handoff.sh` | 设计交接，带 SHA256 指纹 |
| `comet-archive.sh` | 一键归档 |
| `comet-state.sh` | 统一状态管理（读写 YAML，防 AI 直接编辑出错） |
| `comet-yaml-validate.sh` | YAML 结构校验 |
| `comet-hook-guard.sh` | PreToolUse hook：在 open/design/archive 阶段禁止写代码 |
| `comet-env.sh` | 脚本发现 |

**设计哲学**：AI 擅长生成内容，但状态管理/条件判断/格式校验易出错。硬逻辑交给脚本，创造交给 AI。

## 五、核心价值

1. **自动化编排** — 一个命令从想法到归档，告别手动阶段管理
2. **中断恢复** — `.comet.yaml` 状态机让换终端/重启不会丢失上下文
3. **强制纪律** — verify 阶段强制执行不允许"差不多就得"
4. **Skill 嵌套** — 展示了底层 Skill 如何被自动触发、Skill 链如何自动流转
5. **从"写得好"到"每次都能写得好"** — 编排层的核心价值

## 六、相关链接

- GitHub：https://github.com/rpamis/comet
- 原文：https://mp.weixin.qq.com/s/1rWfugeHVB5ETnJZFpufxw
