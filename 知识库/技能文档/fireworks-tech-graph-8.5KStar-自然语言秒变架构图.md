---
tags:
  - 架构图
  - SVG
  - UML
  - Claude Code Skill
  - 自然语言出图
source: 微信公众号 开源星探
author: 开源星探
date: 2026-07-12
status: completed
---

# fireworks-tech-graph 8.5K⭐：自然语言秒变专业技术架构图

> 公众号 **开源星探** · 2026年7月12日
> GitHub: [yizhiyanhua-ai/fireworks-tech-graph](https://github.com/yizhiyanhua-ai/fireworks-tech-graph)

## 核心能力

专为 AI 编码助手设计的 Skill，自然语言描述 → 自动输出 SVG + PNG。

```
用户输入："帮我画一张 Mem0 内存架构图，用深色风格"
→ Skill 自动分类：内存架构图，风格2（Dark Terminal）
→ 生成带泳道、圆柱、语义箭头的 SVG
→ 导出 1920px 高清 PNG
→ 输出：mem0-architecture.svg / mem0-architecture.png
```

## 8 种视觉风格

| 风格 | 特点 | 适用场景 |
|------|------|---------|
| Flat Icon（默认） | 白底，彩色强调色，扁平图标 | 技术博客、文档 |
| Dark Terminal | 深色背景，霓虹配色，等宽字体 | GitHub README、开发者文章 |
| Blueprint | 深蓝底，网格线，青色描边 | 正式架构设计文档 |
| Notion Clean | 极简白底，单色箭头，中性边框 | Notion、Confluence Wiki |
| Glassmorphism | 深色渐变背景，磨砂玻璃卡片 | 产品官网、演讲Keynote |
| Claude Official | 温暖奶油色，Anthropic 品牌色 | Anthropic 相关文档 |
| OpenAI Official | 纯白背景，OpenAI 品牌配色 | OpenAI 相关文档 |
| **Dark Luxury** (AI手绘) | 深黑背景，香槟金点缀，衬线标题 | 高端展示型架构图 |

> Style 8 Dark Luxury 与众不同——不是基于模板，而是让 AI 读取参考文档后**直接手绘 SVG**，呈现出独特的艺术感。

## 14 种 UML 图类型覆盖

**结构类：** 类图、组件图、部署图、包图、组合结构图、对象图
**行为类：** 用例图、活动图、状态机图、序列图、通信图、时序图、交互概览图
**数据类：** ER 图

## AI/Agent 领域内置模式

- RAG 流程（向量检索、嵌入生成、上下文构建、重排序）
- Agentic Search（智能体搜索、工具调用、结果聚合）
- Mem0 记忆架构（向量存储、图数据库、KV 存储、历史存储）
- Multi-Agent（协调器、专家智能体、共享记忆、结果合成）
- Tool Call 流程（工具选择、执行、解析、循环反馈）

## 语义形状与箭头系统

| 形状 | 含义 |
|------|------|
| 双边框圆角矩形 | LLM / 模型 |
| 六边形 | Agent / 协调器 |
| 带网格线的圆柱体 | 向量存储 |
| 齿轮图标矩形 | 工具 / 函数 |
| 虚线圆角矩形 | 短期记忆 |
| 数据库圆柱体 | 长期记忆 |
| 小人图标 | 用户 / 人类 |
| 水平管道 | 队列 / 流 |

| 箭头语义 | 含义 |
|---------|------|
| 蓝色实线 | 主数据流 |
| 橙色 | 控制流 |
| 绿色实线 | 内存读取 |
| 绿色虚线 | 内存写入 |
| 紫色曲线 | 反馈循环 |
| 灰色虚线 | 异步通信 |

## 安装

```bash
# 安装 Skill
npx skills add yizhiyanhua-ai/fireworks-tech-graph

# 或克隆到本地
git clone https://github.com/yizhiyanhua-ai/fireworks-tech-graph.git ~/.claude/skills/fireworks-tech-graph

# 安装 PNG 渲染依赖（三选一）
pip install cairosvg           # 推荐
brew install librsvg           # macOS 备选
npm install puppeteer          # 最高保真

# 更新技能
npx skills add yizhiyanhua-ai/fireworks-tech-graph --force -g -y
```

**触发关键词：** `generate diagram` / `draw diagram` / `create chart` / `visualize` / `architecture diagram` / `flowchart` / `sequence diagram` / `data flow`

## 40+ 产品图标

内置主流技术产品图标并还原品牌配色：
- AI 模型：OpenAI、Anthropic、Google、Meta
- 向量数据库：Pinecone、Weaviate、Milvus、Chroma
- 数据库：PostgreSQL、MySQL、MongoDB、Redis
- 消息队列：Kafka、RabbitMQ
- 云服务：AWS、GCP、Azure

---

**对比已有归档：** 知识库已有 [[Archify Skill-Agent直接出架构图]]（07-11归档），两者定位不同——Archify 侧重 UML/流程图/时序图，fireworks-tech-graph 侧重 AI/Agent 领域架构图 + 8种风格系统 + 语义形状体系。
