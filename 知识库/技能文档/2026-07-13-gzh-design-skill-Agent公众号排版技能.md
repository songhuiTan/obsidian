# gzh-design-skill：专为 AI Agent 打造的公众号排版技能

> 作者：AI开源提效指南
> 来源：微信公众号「AI开源提效指南」
> 日期：2026-07-13 06:35
> 原文：[gzh-design-skill：专为 AI Agent 打造的公众号排版技能](https://mp.weixin.qq.com/s/FYOMmtcgWpCi1RNeV3DQ3Q)
> GitHub：https://github.com/isjiamu/gzh-design-skill

---

## 简介

gzh-design-skill 是一个专为 AI Agent（Claude Code / Codex / Cursor 等）打造的**公众号排版 Skill**。上线不到两周 Star 突破 2000。

**核心理念：** Markdown → 一键变成可直接粘贴进微信公众号编辑器的精致 HTML。终结"排版两小时，粘贴全白费"。

---

## 核心亮点

| 特性 | 说明 |
|------|------|
| 粘贴不掉格式 | 全内联样式 + `<span leaf="">` 包裹，专避公众号过滤规则 |
| 模型无关 | 排版逻辑全在组件库和脚本，Claude / GPT / 国产模型效果一致 |
| 组件库驱动 | 每套主题是一个独立组件库文件，AI 从组件库取 HTML 不凭记忆手写 |
| 双关卡校验 | `component_lint.py`（源头）+ `validate_gzh_html.py`（产物），两关全绿才交付 |
| 主题生成器 | 一句话或一张参考图 → AI 自动生成完整主题组件库 |

---

## 6 套精选主题

| 主题 | 风格 | 适用场景 |
|------|------|----------|
| **摸鱼绿（默认）** | 卡片丰富，信息密度高 | 教程、测评、清单、工具盘点 |
| **红白色系** | 经典编辑风 | 深度分析、观点、力量感话题 |
| **石墨极简风** | 极简克制 | 设计、科技评论、专业观点 |
| **留白禅意风** | 呼吸感最强 | 禅意、极简生活、深度随笔 |
| **摸鱼票据风** | 票据视觉隐喻 | 工具对比、创意评测 |
| **橄榄手记** | 编辑部内刊质感 | 内刊手记、深度评测、案例复盘 |

---

## 处理流程

```
用户提供 Markdown
    → ① 选主题（自动推荐）
    → ② 读组件库（主题库 + 通用增量库）
    → ③ 解析 Markdown 结构
    → ④ 按配方选组件 → 装配 HTML
    → ⑤ 校验合规（双关卡）
    → ⑥ 输出：干净正文 + 带复制按钮的预览页
```

### 智能排版细节
- **章节自动编号** — `##` 顺序分配 01/02/03…，末章用 `∞` 或 `///`
- **关键词下划线** — 每段主动标 1-3 个关键词
- **中文全角标点** — 正文自动规范全角，代码块原样保留
- **尾部签名区** — 去重合并，默认用占位符

---

## 安装与使用

```bash
# 方式一：一行安装（推荐）
npx skills add https://github.com/isjiamu/gzh-design-skill

# 方式二：手动 clone
git clone https://github.com/isjiamu/gzh-design-skill.git ~/.claude/skills/gzh-design
```

**使用：** 装好后对 Agent 说 → `用摸鱼绿把这篇文章排成公众号 HTML：article.md`

### 手动校验
```bash
python3 scripts/component_lint.py .                          # 源头关
python3 scripts/validate_gzh_html.py 文章_排版_摸鱼绿.html   # 产物关
```

### Word 文档转换
```bash
python3 scripts/extract_docx.py 文章.docx -o 文章.md
# 然后让 AI 排版
```

---

## 组件库体系（项目结构）

```
references/
├── theme-index.md               # 主题索引（单一来源）
├── theme-moyu-green.md          # 摸鱼绿组件库（936 行）
├── theme-red-white.md           # 红白色系
├── theme-graphite-minimal.md    # 石墨极简风
├── theme-zen-whitespace.md      # 留白禅意风
├── theme-moyu-ticket.md         # 摸鱼票据风
├── theme-olive-journal.md       # 橄榄手记
├── common-components.md         # 通用增量库
├── theme-generator.md           # 主题生成器工作流
├── format-normalize.md          # 格式归一化
└── eval-cases.md                # 触发用例
```

每个主题库含 5 个必备章节：设计变量速查表 → 各组件 HTML → 文章模板骨架 → 文章类型→组件配方表 → Markdown→组件映射规则。

---

## 平台红线（核心规则）

| 禁止 | 必须 | 可用 |
|------|------|------|
| `<style>/<script>/<div>` | 样式全部内联 `style` | `display:flex`（有限） |
| `class`/`id` | 所有文字节点用 `<span leaf="">` 包裹 | `linear-gradient` |
| `position:fixed/absolute/sticky` | | `border-radius`、`box-shadow` |
| `float`、`@media`、`@keyframes` | | `<section>/<p>/<span>/<strong>/<img>/<h3>` |
| `display:grid`、CSS 变量、外部字体 | | |

---

## 归档信息

- 公众号：AI开源提效指南
- 归档日期：2026-07-13
- 原文链接：https://mp.weixin.qq.com/s/FYOMmtcgWpCi1RNeV3DQ3Q
- GitHub：https://github.com/isjiamu/gzh-design-skill
