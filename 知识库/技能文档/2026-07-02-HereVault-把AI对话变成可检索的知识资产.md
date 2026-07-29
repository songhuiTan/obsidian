# HereVault：把 AI 对话变成可检索的知识资产

> 来源：公众号「浴霸兄」（作者：浴霸兄）
> 时间：2026-06-08
> 归档：2026-07-02

## 一、核心定位

面向 AI Agent 的本地记忆与知识库系统，基于 **Obsidian Vault** 构建。

**"对话即知识，知识即资产"**

解决的核心问题：AI 对话"聊完即丢"，对话成果（方案讨论、bug 排查、技术选型）没有沉淀机制；已有的笔记无法被 AI 检索调用。

## 二、技术架构

```
Obsidian Vault (Markdown 文件)
       │ 监听文件变化
       ▼
HereVault Server
  ├── Embedder → LanceDB (BGE-M3 ONNX int8 量化)
  └── SearchEngine (向量 + BM25 + Reranker)
       │
   ┌───┼───┐
   ▼   ▼   ▼
  MCP  HTTP  CLI
```

## 三、核心能力

| 能力 | 实现 | 价值 |
|------|------|------|
| 记忆管理 | 六类记忆分类 | 沉淀 AI 交互成果，跨会话复用 |
| 知识库 | BGE-M3 嵌入 + LanceDB | 让静态笔记可被语义检索 |
| 混合搜索 | 向量 + BM25 + Reranker | 召回率 85%+，精准度 90%+ |
| 本地优先 | 数据全在本地 | 数据可控，无云端依赖 |
| 多端接入 | MCP / HTTP API / CLI | 适配不同集成场景 |

## 四、记忆系统：六种类型

- **conversation** — 对话讨论与结论
- **fact** — 项目事实与决策
- **context** — 任务上下文
- **skill** — 工作流与技能
- **preference** — 用户偏好
- **habit** — 行为习惯

记忆以 Markdown + YAML frontmatter 格式保存到 Vault，可直接在 Obsidian 查看编辑，支持 `[[双链]]` 关联。

## 五、混合搜索管线

```
用户查询
  ↓
向量检索 (BGE-M3 编码) → Top-50
BM25 关键词匹配      → Top-50
  ↓
RRF 融合
  ↓
Reranker 精排 → Top-5 高质量结果
```

## 六、Vault 目录结构

```
your-vault/
├── .herevault/       # 系统数据（配置、模型、向量库）
├── Memories/         # 记忆存储（按类型分子目录）
│   ├── conversation/
│   ├── fact/
│   ├── context/
│   └── ...
├── Knowledge/        # 知识库文档
└── Source/           # 待索引的源文件
```

## 七、性能优化

- **模型懒加载** — Embedder/Reranker 单例，首次使用初始化，2 分钟无请求自动释放
- **后台索引** — 先就绪再异步索引，不阻塞操作
- **查询缓存** — TTL 1 小时，上限 500 条
- **文档去重** — MD5 checksum 跳过未变更文档

## 八、使用方式

**MCP 接入（Cursor/Claude Code）**：
```json
{"mcpServers": {"herevault": {
  "command": "herevault",
  "args": ["serve", "--vault", "/path/to/vault"]
}}}
```

**十分钟上手**：
```bash
npm install -g herevault
herevault init --vault ~/my-vault
herevault download-models --vault ~/my-vault
herevault serve --vault ~/my-vault
```

## 九、设计哲学

1. **数据透明性 > 黑盒优化** — Markdown 为真实数据源，向量库只是索引
2. **本地优先 ≠ 功能阉割** — BGE-M3 ONNX 量化 + LanceDB 嵌入式，本地可达生产级效果
3. **AI 记忆 ≠ 简单日志** — 六类记忆结构化，精准检索

## 十、相关链接

- GitHub：https://github.com/stevienichs/herevault
- NPM：https://www.npmjs.com/package/herevault
- 原文：https://mp.weixin.qq.com/s/Y2K_jHlSF53Eal5ghwMk9w
