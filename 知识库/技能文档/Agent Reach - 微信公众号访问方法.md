# Agent Reach - 微信公众号访问方法

## 概述

Agent Reach 是一个让 AI Agent 能够访问 13+ 平台的技能包，包括微信公众号。

## 安装

```bash
pip install https://github.com/Panniantong/agent-reach/archive/main.zip
```

## 微信公众号访问配置

### 依赖安装

```bash
pip install 'camoufox[geoip]' markdownify beautifulsoup4 httpx mcp miku_ai
```

### 安装微信文章工具

```bash
mkdir -p ~/.agent-reach/tools
cd ~/.agent-reach/tools
git clone https://github.com/bzd6661/wechat-article-for-ai.git
```

### MCP 配置

#### 1. 安装 mcporter

```bash
npm install -g mcporter
```

#### 2. 配置微信 MCP 服务器

```bash
~/.npm-global/bin/mcporter config add wechat --command 'python3' --args '["mcp_server.py"]' --cwd ~/.agent-reach/tools/wechat-article-for-ai
```

## 使用方法

### CLI 方式（推荐，稳定）

```bash
cd ~/.agent-reach/tools/wechat-article-for-ai
python3 main.py "https://mp.weixin.qq.com/s/文章ID"
```

### MCP 方式（通过 mcporter）

```bash
# 读取单篇文章
~/.npm-global/bin/mcporter call 'wechat.convert_article(url: "https://mp.weixin.qq.com/s/文章ID")'

# 批量转换文章
~/.npm-global/bin/mcporter call 'wechat.batch_convert(urls: ["url1", "url2", "url3"])'
```

## 工具说明

### wechat-article-for-ai

**特点：**
- 使用 Camoufox（隐身 Firefox）绕过微信反爬虫
- 自动下载图片到本地
- 保留代码块格式
- 输出 Markdown + YAML frontmatter
- 支持 MCP 服务器

**输出结构：**
```
output/
  <文章标题>/
    <文章标题>.md
    images/
      img_001.png
      img_002.jpg
```

## 注意事项

1. **首次运行**：Camoufox 浏览器需要下载（约 306 MB），会比较慢
2. **超时问题**：MCP 调用可能超时，建议使用 CLI 方式
3. **反爬虫**：微信有反爬机制，频繁请求可能触发验证码
4. **路径问题**：确保使用 `python3` 而不是 `python`

## 相关链接

- Agent Reach: https://github.com/Panniantong/agent-reach
- wechat-article-for-ai: https://github.com/bzd6661/wechat-article-for-ai
- mcporter: npm 包管理器

## 配置文件位置

- mcporter 配置: `~/.openclaw/workspace/config/mcporter.json`
- 微信工具: `~/.agent-reach/tools/wechat-article-for-ai`

---

*更新日期: 2026-03-09*
