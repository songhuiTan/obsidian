---
title: "Ponytail：让AI编码Agent像最懒资深工程师一样思考"
source: "Dietrich Gebert"
source_url: "https://github.com/DietrichGebert/ponytail"
date: "2026-06-12"
tags: [AI, Agent, Skill, 编码, YAGNI, 开源]
---

# Ponytail：让AI编码Agent像最懒资深工程师一样思考

> GitHub：[DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail)（142.8K⭐ / 7.7K fork / MIT，2026-06-12 发布）
> 官网：https://ponytail.dev
> 作者：Dietrich Gebert

## 概述

**Ponytail** 是一个 Agent Skill 集合，核心思想一句话：**最好的代码是没写出来的代码**。它让你的 AI 编码 Agent（Claude Code / Codex / Hermes / Cursor 等 20+ 宿主）进入"全屋最懒的资深工程师"模式——先质疑任务是否需要存在（YAGNI），再复用已有的东西，最后才写最少量的代码。

不是让 AI 笨，而是纠正当前 Agent 的系统性偏向：**默认过度生成**。

## 核心数据（README 口径）

| 指标 | 数值 |
|------|------|
| 代码量 | 平均 **-54%**（过度设计场景最高 -94%） |
| 成本 | **-20%** |
| 速度 | **+27%** |
| 安全护栏 | 100% 保留（对照组裸"写一行"提示词会丢安全护栏） |
| 基准方法 | 真实 Claude Code 会话，Haiku 4.5，12 个特性任务，n=4 |

对比案例（同任务同模型，无 skill vs 有）：Email 校验 75→3 行，Debounce 116→10 行，倒计时 267→9 行。

早期宣传的 80-94% 是单次基准的区间上限；对着公平的 agentic 基线，94% 是逐任务上限而非平均——作者主动在 README 修正过这一点。

## 懒人阶梯（核心机制）

每一步先问，停在最先成立的台阶上：

1. **这任务需要存在吗？** 投机需求 = 跳过，一行说明（YAGNI）
2. **代码库里已经有了？** 复用，别重复造几步之外就有的轮子
3. **标准库能做？** 用标准库
4. **平台原生特性覆盖？** `<input type="date">` 优于日期库，CSS 优于 JS，DB 约束优于应用层校验
5. **已装的依赖能解？** 用它，不为几行代码引入新依赖
6. **能一行写完？** 一行
7. **以上都不行：** 写能工作的最少代码

两条硬规则：**Bug 修复 = 根因修复**（先 grep 所有调用方，在汇聚点修一次，别在每个调用点打补丁）；**删除优于添加**。

## 六个 Skill

| Skill | 用途 |
|-------|------|
| ponytail | 主模式：懒人阶梯 + lite/full/ultra 三档 |
| ponytail-review | Diff 审查：只找过度设计——重复造轮子、投机抽象、死灵活性 |
| ponytail-audit | 全仓审计：可删/可简化/可换标准库的排序清单 |
| ponytail-debt | 技术债台账：收割代码里的 `ponytail:` 注释成追踪清单 |
| ponytail-gain | 收益计分板：展示基准中位数的省码省钱数据 |
| ponytail-help | 快速参考卡 |

## 安装方式（Hermes 实测）

**官方插件通道（被安全扫描拦截）：**
```bash
hermes plugins install DietrichGebert/ponytail --enable
```
实测 2026-09：被 Hermes 插件扫描器判 DANGEROUS 拦截——97 条 findings 全来自 `benchmarks/` 跑分目录的字面量（如测试路径穿越防护用的 `../../etc/passwd` 字符串），属误判，但 dangerous 判定不可 --force 覆盖。

**手动安装 skill（本机实际采用的路线，已验证）：**
```bash
git clone --depth 1 https://github.com/DietrichGebert/ponytail.git /tmp/ponytail
# 只拷 skills/，不碰 benchmarks/
cp -r /tmp/ponytail/skills/ponytail{,-audit,-debt,-gain,-help,-review} ~/.hermes/skills/
hermes skills list | grep ponytail   # 6 个全部 enabled
```
skill 文件为纯 markdown 规则，零网络/执行调用，已逐文件核查。生效需新会话；`/ponytail lite|full|ultra` 切档，"stop ponytail" 退出。

其他宿主（Claude Code / Codex / ChatGPT / Copilot / Cursor / Gemini CLI / Swival 等）见仓库 `docs/agent-portability.md` 的适配矩阵。

## 战略分析

### 与现有体系的对照

| 维度 | Ponytail | no-ai-slop / Stop Slop | Chisle |
|------|----------|------------------------|--------|
| 纠正对象 | Agent 编码过度生成 | 文字 AI 腔 | 三轴：简洁措辞+YAGNI+上下文 |
| 形式 | 6 skill + 多宿主适配 | 单 SKILL.md | 单仓库 |
| 实证 | 真实会话基准，-54% 代码均值 | 20+ 模式证据清单 | 模仿 Ponytail 加措辞轴 |
| 热度 | 142.8K⭐ 三个月 | 3.7K~13K⭐ | 477⭐ |

"反过度生成"已成一条独立赛道：文字侧（no-ai-slop/Stop Slop）、前端侧（taste-skill）、编码侧（Ponytail）。共同点都是**用可执行的判据替代形容词**——不写"要简洁"，写"停在哪一阶阶梯"。

### 高价值洞察

1. **基准的自我修正姿态**：作者主动把早期 80-94% 宣传数据修正为"逐任务上限而非平均"，并公开复现脚本。在全网晒跑分的氛围里，这种口径诚实反而强化可信度。
2. **Hermes 插件扫描器需要 benchmark 目录豁免**：测试安全用例字面量触发 traversal 判定，误伤所有带安全测试的正规仓库。对含 benchmark 目录的仓库，手动装 skills/ 子目录是干净替代路线。
3. **与 Hermes 的适配已官方化**：v4.8.4 起原生支持（always-on context + 斜杠命令），说明主流 Agent 框架正在把"节制品格"作为一等公民接入。

### 差距与借鉴点

- Ponytail 不处理**输出措辞**（语气词、废话）——那是 no-ai-slop/Chisle 的领域。两者互补：编码任务用 Ponytail 控制代码量，写作任务用 no-ai-slop 控制行文。
- `ponytail-debt` 的"标记-追踪"闭环设计值得借鉴：每个故意走捷径处留 `ponytail:` 注释，定期收割成台账——把"以后再说"变成可审计的债。

## 关联阅读

- [[2026-08-02-no-ai-slop-去除AI味写作Skill]]
- [[2026-06-10-昨天GitHub被AI屠榜了但这波风向变了]]
- [[Prompt-Engineering_Skill-蒸馏5家系统提示词]]
