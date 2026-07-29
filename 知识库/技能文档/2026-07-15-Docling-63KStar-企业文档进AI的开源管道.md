---
title: "63K Star！企业文档进 AI 前，最值钱的脏活被这个开源项目接住了"
source: "风筝手札"
source_url: "https://mp.weixin.qq.com/s/vmlgwX6TrIMdu92gwJWimA"
date: "2026-07-15"
tags: [Docling, 文档解析, OCR, RAG, 开源, PDF]
---

> 企业做知识库最耗时的不是聊天框，而是文档处理。Docling 把 PDF/DOCX/PPTX/XLSX/HTML/EPUB/邮件/图片/音频/LaTeX 解析成 AI 可消化的结构化内容。

项目地址：[docling-project/docling](https://github.com/docling-project/docling)（63K+ ⭐，MIT License）

## 为什么文档解析是关键瓶颈

企业文档太脏——合同 PDF、财报 XBRL、培训 PPT、扫描件、邮件附件。RAG 前面最耗时的不是聊天框，而是文档处理：

- PDF 双栏排版 → 阅读顺序乱
- 表格跨页 → 抽取变形
- 扫描件需 OCR
- 财报/专利/合同有自身结构

**如果前面的解析做不好，后面的 RAG 再漂亮也没用。** 模型拿到碎掉的表格、乱序的段落、缺失的标题，自然答偏。

Docling 不做完整应用，而是做企业文档的**入口清洗层**。

## 能力覆盖

**输入格式**：PDF、DOCX、PPTX、XLSX、HTML、EPUB、WAV、MP3、WebVTT、EML、MSG、PNG、TIFF、JPEG、LaTeX、纯文本

**PDF 处理**：版面分析、阅读顺序、表格结构、代码、公式、图片分类

**输出格式**：Markdown、HTML、WebVTT、DocLang、DocTags、lossless JSON

**专用结构**：USPTO 专利、JATS 学术文章、XBRL 财务报告

## 快速上手

```bash
pip install docling
docling https://arxiv.org/pdf/2206.01062
```

Python API：
```python
from docling.document_converter import DocumentConverter
source = "contract.pdf"
converter = DocumentConverter()
result = converter.convert(source)
print(result.document.export_to_markdown())
```

还提供 MCP server 和 API server，可接入 Agent 或独立部署。

## 商业落地方向

1. **企业知识库文档清洗服务** — 批量解析、结构化、去噪、转 Markdown/JSON
2. **合同和标书解析** — 解析层 + 条款抽取/风险识别/差异对比
3. **财报和行业报告入库** — XBRL 支持，金融/投研刚需
4. **专利和论文数据库** — USPTO/JATS 结构支持
5. **私有化文档 AI 管道** — 本地执行，适合银行/律所/政企

## 与普通 PDF 转 Markdown 工具的区别

普通工具解决"能不能转"，Docling 解决"转出来能不能用"——保留标题层级、表格结构、段落顺序、公式和代码完整，文档来源可追踪。

## 注意边界

- 代码 MIT，但模型依赖许可证要单独看
- 不同企业的文件模板/扫描质量/语言/表格复杂度不同，交付需模板适配 + 异常处理 + 人工校验

---

## 战略分析

**Docling 与 kb-archive-article 流水线直接相关。** 我们目前的归档流程主要处理微信公众号文章（HTML），但如果遇到 PDF/扫描件/EPUB 等格式的内容，Docling 可以作为解析层前置处理。

**与现有体系的对照：**
- 归档流程目前用 Firecrawl（HTML）+ 有赞 parser（WeChat），缺少 PDF/扫描件/多格式文档的解析链路
- Docling 的 lossless JSON 输出比 Markdown 更适合结构化存储
- MCP server 接入方式可以直接作为 Hermes 的一个 Tool，让 Agent 在需要时调用解析文档

---

## 归档日志

- 2026-07-16 归档
