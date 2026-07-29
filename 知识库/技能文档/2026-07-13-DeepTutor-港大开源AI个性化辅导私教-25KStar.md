# 25k Star，港大开源了一款 AI 个性化辅导私教：DeepTutor

> 作者：小黑
> 来源：微信公众号「极客之家」
> 日期：2026-07-13 03:19
> 原文：[25k Star，港大开源了一款 AI 个性化辅导私教，在 GitHub 上杀疯了！](https://mp.weixin.qq.com/s/MfTnEwjQlBJX4bf0JsqbRw)
> GitHub：https://github.com/HKUDS/DeepTutor

---

## 简介

DeepTutor 是香港大学数据科学实验室开源的 AI 学习工作空间。将 Chat、Quiz、Research、Solve、Visualize、Mastery Path 六种学习模式塞进同一个 Agent 引擎，数据在所有工作流里共享。开源约 100 天冲到 25k Star。

- **Agent 循环写到底层**，六种模式共享一个 runtime
- 支持本地模型（Ollama/LM Studio/llama.cpp/vLLM）+ Docker 一键部署

---

## 功能详情

### 1. Chat：六种模式共享 Agent 引擎
左边导航栏 9 个模块：Home、Partners、My Agents、Co-Writer、Book、Learning Space、Memory、Knowledge Center、Settings。六种学习模式都从 Chat 窗口进入，切模式时上下文不丢失。

### 2. Partners：接入本地 Claude Code / Codex
可在任意对话轮次里接入本地的 Claude Code 或 Codex。Partner 有独立的 Persona、私有知识库和技能，保持独立记忆。对话支持分支、续聊、删除，带可回放的操作轨迹。

### 3. My Agents：自定义 Agent 独立空间
创建和管理自己的 Agent，配不同的 Persona、知识库和技能。Agent 之间记忆隔离，但可通过 Chat 统一调度。

### 4. Co-Writer：多文档协同写作
同时打开多个文档，AI 根据知识库内容辅助写作。支持智能编辑、自动标注和 TTS 朗读，可直接保存到笔记本或导出 Markdown。

### 5. Book：活书编译器
将笔记和对话内容编译成 HTML 书籍。左章节导航、右内容区，支持文本、标注、测验、代码、时间线、闪卡、图表、交互式动画和深度探索。每个章节可直接对话提问。

### 6. Knowledge Center：多引擎 RAG + 版本管理
RAG 引擎支持：LlamaIndex、PageIndex、GraphRAG、LightRAG。可链 Obsidian Vault。文档支持 PDF/DOCX/XLSX/PPTX，浏览器直接预览。索引做版本管理，重建不覆盖旧的。

### 7. Learning Space：技能市场与掌握路径
- **Skills 面板**：展示已安装技能，可从 EduHub 导入社区技能
- **Mastery Path**：掌握练习仪表盘，每类题目必须达标才能往下走

### 8. Memory：三层记忆 + Graph 溯源

| 层级 | 说明 |
|------|------|
| L1 | 原始对话 |
| L2 | 摘要 |
| L3 | 综合提炼 |

Memory Graph 能把每条结论追溯到原始证据。三层独立管理，删除某层不影响其他层。v1.4.6 升至顶级导航，随时可查可编辑。

### 9. Settings：统一配置面板
- LLM 提供商：OpenAI、Anthropic、Google、Azure + 本地（Ollama/LM Studio/llama.cpp/vLLM）
- Embedding 可单独配置，不与 LLM 绑定
- 界面主题：深色/浅色，语言：中文/英文

---

## 快速开始

### PyPI 安装
```bash
mkdir my-deeptutor && cd my-deeptutor
pip install -U deeptutor
deeptutor init   # 选端口、LLM提供商、API key、embedding
deeptutor start  # 默认前端3782，后端8001
```

### Docker
```bash
docker run --rm --name deeptutor \
  -p 127.0.0.1:3782:3782 \
  -v deeptutor-data:/app/data \
  ghcr.io/hkuds/deeptutor:latest
```

连接本地 Ollama 需加 `--add-host=host.docker.internal:host-gateway`，Settings 里 Base URL 指向 `http://host.docker.internal:11434/v1`。

### CLI 模式
- `deeptutor chat` — 交互式 REPL
- `deeptutor kb create` — 建知识库
- `deeptutor memory show` — 看记忆状态

---

## 评价

**亮点：**
- Agent 循环写到底层，架构统一
- Partners 能直接 `@Claude Code` 调用本地模型
- RAG 四种引擎可选（但未给出推荐）
- Memory 三层设计 + Graph 溯源

**不足：**
- 迭代太快，文档偶尔落后于代码
- RAG 引擎选择对新手不友好
- 文档对某些配置细节说明不足（如 embedding 模型选错会导致建库报错）

---

## 归档信息

- 公众号：极客之家
- 归档日期：2026-07-13
- 原文链接：https://mp.weixin.qq.com/s/MfTnEwjQlBJX4bf0JsqbRw
- GitHub：https://github.com/HKUDS/DeepTutor
