---
title: "给 AI Agent 加上高质量搜索，实战能力提升 3-4 倍"
source: "https://zcnv48kqce5a.feishu.cn/wiki/TIqIwBHM7i7aYokYZ7kcneuDnPm"
source_platform: "飞书文档"
archived: 2026-11-14
tags: [AnySearch, 搜索, MCP, Hermes, DeepSeek-Harness, 飞书归档]
description: "给 AI Agent 加上高质量搜索，实战能力提升3-4倍：AnySearch 15个一级域42个子域的垂直搜索，MCP/Skill 两种接入方式，额度对比 Tavily"
series: "Hermes&OpenClaw实战进阶配套笔记"
---

# 给 AI Agent 加上高质量搜索，实战能力提升 3-4 倍

> 来源：飞书文档（李君陌AI笔记）｜Hermes & OpenClaw 实战进阶配套笔记 第12篇
> 原文：https://zcnv48kqce5a.feishu.cn/wiki/TIqIwBHM7i7aYokYZ7kcneuDnPm
> 归档时间：2026-11-14

> Agent + 高质量的内容才是核心。本文是「Hermes & OpenClaw 实战进阶配套笔记」中的 AnySearch 专题篇。

## 一、核心结论：换的不是模型，是搜索基础设施

很多人装完本地 Agent，觉得搜索已经很好用了。但实际跑任务会发现：Agent 一旦需要查实时信息、找代码、搜资料，返回的结果质量很差——一堆链接和 HTML，它还得自己爬、自己洗，Token 烧了不少，事还没办成。

给 Hermes 和 DeepSeek Harness 加上 AnySearch 之后，做事效率直接提升 3-4 倍。**并不是换了更强的模型，而是换了一个更好的搜索基础设施。**

### 通用搜索 vs AnySearch

| | 通用搜索 | AnySearch |
|---|---|---|
| 返回物 | 半成品：一堆链接和 HTML，还要自己爬、自己洗 | 在搜索层就把活干完：筛选、排序、去重、结构化，直接返回干净 Markdown 正文，拿到就能用 |
| 垂直信息 | 专业垂直信息搜不到 | 多垂直领域数据源覆盖 |
| 污染 | 被 SEO 和同源内容污染 | 搜索层完成质量过滤 |
| 成本 | 一个任务反复搜 3-4 轮，Token 全浪费在垃圾信息上 | 一轮出干净结果 |

AnySearch 拥有 **15 个一级域 + 42 个子域**的垂直数据源，包括社交媒体、图像搜索、金融、学术、开发、游戏、电影等，极大保证了搜索质量。

### 三个能力跃迁（作者的实测）

1. **图片精准度**：做青岛周末双人游图文攻略时，能把对应配图也精准找到，这对搜索能力要求其实不低。
2. **外网社交媒体**：检索全网关于 DeepSeek Harness 的真实评论，拿到了大量原本拿不到的英文真实评论，扩大了信息边界。
3. **深度资料汇编**：全网检索 DeepSeek Harness 的有价值案例和插件，扩展成一份系统全面的「橙皮书」，有截图有视频，信息源价值含量高。

---

## 二、给 Agent 配置 AnySearch

### 1. 创建 AnySearch 账号

（原文档此处为操作图示）

### 2. 获取 API Key

Key 形如 `as_sk_...`

### 3. 安装 AnySearch

官方文档：https://www.anysearch.com/docs#quick-start

#### （1）给 Hermes 安装

Skill 下载地址：
```
https://github.com/anysearch-ai/anysearch-skill/archive/refs/heads/main.zip
```

也可以采用 **MCP 的安装方式**（对 Hermes 说「帮我添加 anysearch 的 mcp」）：

```json
{
  "mcp": {
    "anysearch": {
      "type": "remote",
      "url": "https://api.anysearch.com/mcp",
      "headers": {
        "Authorization": "Bearer ${ANYSEARCH_API_KEY}"
      }
    }
  }
}
```

#### （2）给 DeepSeek Harness 安装

将插件安装到 `web` profile：

```bash
npx -y @deepseek-ai/dsh plugin --profile web add @anysearch/anysearch-dsh
```

**如果报错了**，先执行这条命令换源：

```bash
pnpm config set registry https://registry.npmjs.org/
```

启动 DeepSeek Harness：

```bash
npx @deepseek-ai/dsh web
```

#### 扩展搜索能力

除 DeepSeek Harness 原生的 `web_search` 外，`@anysearch/anysearch-dsh` 还提供 3 个扩展搜索工具：

- `anysearch_capabilities`：查询当前可用的搜索领域、标签及参数定义
- `anysearch_search`：执行垂直领域或参数化搜索
- `anysearch_batch_search`：一次并发执行 1–5 个独立搜索

通过这些工具，DeepSeek Harness 不仅可以执行常规网页搜索，还可以根据任务类型调用不同的搜索能力。

### 4. 做一次搜索测试

```text
anysearch装好了吗?
```

验证型 prompt：

```text
用anysearch，帮我搜索一些deepseek harness的相关信息，把anysearch的具体执行过程也输出一下，给我做成一份精美的html
```

### 5. 额度说明以及与 Tavily 对比

- **AnySearch**：每天 1000 次额度，学生和开发者认证之后为每日 2000 次
- **Tavily**：每月 1000 次

差距是数量级的——这也是为什么作者建议把 AnySearch 当成 Agent 的默认搜索层。

---

## 三、三个实战案例（可直接复用的 prompt）

### 案例 1：制作青岛周末双人游的图文攻略

```text
使用mcp中的anysearch，帮我制作一份周末去青岛玩两天的图文攻略，图片务必要真实，最后帮我做成一份精美的html
```

**要点**：明确「图片务必要真实」，逼 Agent 走图像搜索域而不是编造图片链接。

### 案例 2：搜索全网对 DeepSeek Harness 的真实评论

```text
帮我用anysearch去找一下关于deepseek harness的真实评论，将所有的真实评论分析并归类一下，并且每个分类下面要把对应的真实评论也放在下面，如果评论是英文，同时翻译一份中文，最终做成一份精美的html
```

**要点**：要求「归类 + 原评论放在分类下 + 英文翻译」，避免 Agent 只给摘要不给原始声音。

### 案例 3：从案例与插件汇编到「DeepSeek Harness 橙皮书」

```text
用anysearch全网检索帮我整理deepseek harness精彩的案例和好用的插件，要图文并茂、不要只有文字，而且图片一定是真实图片，然后将这份资料升级成一份更加细致和详细的deepseek harness橙皮书，内容要更加详细并做成一份html
```

**要点**：从"搜索"升级到"资料汇编成书"，这是 AnySearch 相比通用搜索最大的价值放大点。

---

## 四、作者正在用的 11 个落地场景（养虾养马真正用起来）

> 以下案例在原文档中以录屏演示为主，正文仅保留标题与少量说明。

**案例 0：超级秘书的属性**
一个好的超级秘书一定是主动的——当有跟你相关的重要事件会主动 @你，也会主动问要不要帮你盯着一些事情。

**案例 1：AI 早资讯语音播报版**
每天早晨 8 点推送 AI 资讯，并转换成语音播报，可以在洗漱或路上听。另有一个汇总页面，所有早资讯都会保留下来。

**案例 2：同行与对标直播间监控**
目的是准备做抖音直播，所以要关注同赛道哪些博主做得好、什么形式、讲什么内容，特别是在线人数数据。效果好的直接借鉴，不好的直接避坑，节约试错成本。
> 备注：优化后的版本增加了与作者自身业务相关的建议和分析（需要把"我是做什么的"这些详细信息告诉 Agent）。

**案例 3：直播间挖掘用户真问题需求**
专门盯一个在线观看人数较多的直播间，从中挖掘用户的真实问题需求。

**案例 4：低粉爆款找选题**

**案例 5：评论区问题汇总挖选题（推送链接）**

**案例 6：招聘网站找 B 端客户**

**案例 7：每天定时推送漫画（可定制）单词卡**

**案例 8：电商同行产品负评挖掘分析**

**案例 9：招聘网站挖掘成长行业与项目**

**案例 10：自动检索对标博主新内容的知识点并以 PPT 推送**

**案例 11：定时对标直播间话术改写（完成录制 + 文案提取 + 自动改写）**

---

## 五、可迁移的要点

1. **搜索层质量决定 Agent 上限**。模型不变，只换搜索基础设施就能有 3-4 倍效率差——这是投入产出比最高的一处改造。
2. **MCP 方式比 Skill 包更省事**，一行 remote url + Bearer token 就接上了，Hermes 侧直接说「帮我加 mcp」即可。
3. **prompt 里要写清楚来源约束**（"图片务必真实"、"英文评论同时翻译"），否则 Agent 会用编造内容填补空缺。
4. **让搜索的输出直接进入交付物**（HTML / PPT / 橙皮书），而不是停留在"搜索结果列表"。

---

*来源：飞书知识库「李君陌AI笔记」— Hermes & OpenClaw 实战进阶配套笔记（AnySearch 篇，最后修改 2026-09-05）*
*原文链接：https://zcnv48kqce5a.feishu.cn/wiki/TIqIwBHM7i7aYokYZ7kcneuDnPm
