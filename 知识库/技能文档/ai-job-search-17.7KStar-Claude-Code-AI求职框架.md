# ai-job-search：17.7K Star Claude Code AI 求职框架

> 来源：开源星探 · 2026-07-09
> 原文：https://mp.weixin.qq.com/s/yv6RhF2ZNElTIjA5GjmPJA
> 分类：Claude Code & Codex

---

GitHub 趋势榜登顶项目，1 日增长超 3.7K Star，累计 17.7K Star。基于 Claude Code 构建的 AI 求职框架，从评估岗位匹配度、定制简历、写求职信到面试准备，全流程自动化。

---

## 项目简介

开发者 MadsLorentzen 创建，基于 Claude Code 的 AI 求职应用框架。Fork 项目、填好个人资料，剩下交给 Claude 处理——评估工作机会、定制简历、撰写求职信、准备面试。

核心工作流：

```
/setup → /scrape → /apply <url>
   |        |          |
   v        v          v
 填写个人资料  搜索岗位并匹配  评估匹配度并生成申请材料
   |        |          |
   v        v          v
 生成结构化档案  按匹配度排序展示  起草+审阅双引擎优化
```

---

## 核心亮点

### 1. 智能画像系统

运行 `/setup` 命令引导建立个人档案，三种方式：

- **文档导入模式**：将简历、领英导出、学历证书、推荐信放入 `documents/` 文件夹，自动读取并整理为结构化能力模型
- **CV 粘贴模式**：直接粘贴单个简历文本，自动解析
- **引导式访谈模式**：像面试官一样聊天，逐步挖掘经历和技能

### 2. 岗位搜索与智能匹配

运行 `/scrape` 命令，从各大招聘平台抓取职位信息，根据个人画像进行匹配度打分。从技能、经验、文化契合度、地理位置、职业发展等多维度综合评估，匹配度最高的岗位排在最前。

### 3. 起草 + 审阅双引擎工作流

`/apply` 命令启动"起草者-审阅者"双引擎工作流：

1. **起草引擎**：根据岗位描述量身打造简历和求职信
2. **审阅引擎**：研究目标公司，对初稿挑刺提意见
3. **修改优化**：起草引擎根据审阅反馈修改完善

有效避免 AI 生成内容常见的"自嗨"问题。

### 4. PDF 自动编译与视觉检查

AI 把 LaTeX 编译成 PDF 并"阅读"渲染效果。发现简历超页、排版错乱、字体不一致等问题自动修复重排，直到简历严格控制在两页、求职信一页。

### 5. 技能缺口分析与学习规划

- `/expand`：扫描公开资料（GitHub、作品集、竞赛主页），挖出文档没写但实际具备的技能，补充进档案
- `/upskill`：对比能力和目标岗位要求，生成技能缺口热力图 + 学习计划 + 资源推荐

### 6. 薪资参考工具

内置薪资查询工具，导入搜集的薪资数据（工会统计、Glassdoor 等），投递前查市场行情。

---

## 快速上手

```bash
# Fork 项目
gh repo fork MadsLorentzen/ai-job-search --clone
cd ai-job-search

# 安装职位搜索 CLI 工具依赖
cd .agents/skills/jobbank-search/cli && bun install && cd ../../../..
cd .agents/skills/jobdanmark-search/cli && bun install && cd ../../../..
cd .agents/skills/jobindex-search/cli && bun install && cd ../../../..
cd .agents/skills/jobnet-search/cli && bun install && cd ../../../..

# 启动 Claude Code
claude

# 在 Claude Code 中运行
/setup    # 设置个人资料
/scrape   # 搜索匹配岗位
/apply <url>  # 申请职位
```

---

## 关键信息

**GitHub**：https://github.com/MadsLorentzen/ai-job-search

**技术栈**：Claude Code + LaTeX + Bun（职位搜索 CLI）
