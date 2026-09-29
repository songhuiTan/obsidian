---
tags: [AI, DSH, DeepSeek-Harness, Ontology, 本体论, Agent数据治理, 插件, 实测, 视频归档]
---

# dsh-ontology 实测：给 Agent 记忆装上本体约束

> 来源：心晴九天·本体论以及企业AI实战 第2集｜抖音 2026-09-11 11:39
> 视频：https://www.douyin.com/video/7684117328598548709
> GitHub：[tancheng33/dsh-ontology](https://github.com/tancheng33/dsh-ontology)（14⭐ / TypeScript / MIT / 2026-08-16 创建，社区插件非 DeepSeek 官方）
> 标签：AI, DSH, Ontology, 本体论, Agent数据治理, 视频归档

---

## 概述

心晴九天实测 **dsh-ontology**——DeepSeek Harness（DSH）的类型化本体插件，核心主张一句话：**Agent 记忆不该是一袋字符串，写入的事实必须先过本体模型校验，业务事实的底线由工具层兜住，不靠模型自觉。**

作者 tancheng33，TypeScript，MIT。与视频同期（8月）后基本停更（最后 push 2026-08-16），属于个人研究型项目。

## 实测验证的核心场景（视频内容）

**报销单"当前审批人"唯一性约束**：
1. 定义关系：报销单的"当前审批人"为 functional（唯一）关系
2. 第一次写入李四 → 成功
3. 同一张报销单写入王五 → **插件直接拦截拒绝**，查询结果仍保留李四
4. 必须先显式撤回旧事实（retract 李四），王五才能写入

**48 项测试全通过**：插件自带 47 项单测（规则/工具/prompt 段落）+ 作者新增的这项业务测试。

作者的框架：**DSH 管运行边界（运行环境、工具、数据链路可控），Ontology 管语义边界（业务事实不被模型偏差污染）**。模型允许判断有偏差，但偏差绝不能直接落进核心数据。

作者同时清醒地划界：实测仅证明工具层约束有效，距离权限、审计、并发和生产级治理还有距离。插件无本体可视化编辑器（只能工具定义和查询，不能拖拽画图）。

## 插件机制（README 全文精读）

### 四个工具
```
ontology_define   声明类和类型化关系（TBox）
ontology_assert   记录实体和事实（ABox，校验后落盘）
ontology_query    查询、遍历、推理
ontology_retract  撤回，带依赖保护
```

### 与笔记式记忆的对比（README 核心表）

| | 笔记式记忆 | dsh-ontology |
|---|---|---|
| 结构 | 自由文本 | 类、类型化关系、实体、三元组 |
| 坏输入 | 静默存储 | **拒绝并指名违反了哪条约束** |
| 隐含事实 | 手工重述，逐渐漂移 | 从 transitive/symmetric/inverseOf **读时推导** |
| 撤回 | 孤儿文本残留 | 随实体级联，推导事实随前提消失 |
| 读回 | grep 碰运气 | 按 class（子类感知）/三元组模式/邻域/最短路径查询 |

### 执行的规则
- **子类包含**：subClassOf 传递；定义时拒绝环
- **Domain/Range**：关系两端类型约束，空列表 = 诚实的"未决定"而非静默全放行
- **Functional 基数**：functional 关系每主语至多一对象；第二个不同对象 = functional-conflict，**拒绝而非静默覆盖**（同对象重述保持幂等）——这正是视频实测的"李四/王五"场景
- **蕴含**：transitive/symmetric/inverseOf 推导至不动点；**推导事实读时派生、永不落盘**，撤回前提即全部消失，每条带 via 标注规则来源
- **撤回安全**：撤实体级联其所有事实；撤术语时若有依赖（还有实体用它/还出现在关系签名里）则拒绝并点名依赖方

### 工程细节
- `dsh plugin --profile <name> add dsh-ontology` 一键装；自带存储栈（dsh-storage 系），数据落 `$DSH_HOME/storages/<domain>.json`
- 配置项齐：strict（false 时记录违规但存，用于探索期建模）、inference、promptSection（词表注入系统提示词，上限 promptMaxTerms=60）、容量护栏 maxEntities=20000 / maxFacts=100000
- 推理核心是**纯函数**（无 IO/存储/Cordis 依赖），可脱离 harness 独立用：`import { entail, findPath, validateFactInput } from 'dsh-ontology/ontology'`
- 要求 DSH >= 0.1.0-rc.6、Node ^22.19 || >=24

## 战略分析

### 与知识库现有体系的对照

| 维度 | dsh-ontology | 知识库相关归档 | 说明 |
|---|---|---|---|
| 宿主 | DeepSeek Harness 插件 | [[技能文档/2026-09-02-DeepSeek-Harness缓存优化-源码拆解]] | 同一生态的缓存与语义两篇 |
| 企业定位 | DSH 平台 + 本体治理 | [[技能文档/2026-09-16-MOVO-基于DeepSeek-Harness的企业Agent平台]] | MOVO 是平台化路线，这是插件路线 |
| "拦住坏写入" | functional 冲突拒绝 | [[技能文档/Skill Quality Gate体系-别让不稳定的AI能力进入团队复用]] | 一个守技能质量，一个守数据事实 |
| 图谱记忆 | TBox+ABox 三元组 | [[技能文档/cross-tool-chat-indexer-AI-Agent共享上下文]] | indexer 管共享上下文，ontology 管事实约束 |

### 值得借鉴的点

1. **"约束即记忆"的范式**——Agent 记忆的正确演进方向不是更大的向量库，而是带 schema 的受控写入。拒绝时指名违反哪条约束，这个信号对模型自纠错极有价值（free-text 记忆给不出）
2. **推导读时派生、永不落盘**——蕴含事实不占存储、撤回前提自动消失，这个设计比"物化所有推导"干净得多
3. **纯函数推理核心可独立复用**——entail/findPath/validateFactInput 不绑 DSH，任何 TS 项目可直接引入做本体校验层
4. **视频方法论本身值得学**——不讲概念直接实测：定义唯一关系 → 制造冲突 → 验证拦截 → 补业务测试进测试套件（47+1=48）。企业 AI 内容的信任来自可复现的验证，不是 PPT
5. **作者的边界意识**——明确"社区插件非官方"、"工具层约束 ≠ 生产治理"，这种划界本身就是企业级思维

### 风险与边界

- 14⭐ 个人项目，2026-08-16 后未再更新，勿直接上生产
- 无可视化本体编辑器，建模全靠 JSON 工具调用，门槛高
- 单机 JSON 存储（$DSH_HOME/storages/），无并发/事务/审计——作者自己也承认
- 本体建模成本前置：类和关系定义错了，约束反而挡住正确数据（strict=false 的探索模式是缓解）

## 归档日志

- 2026-09-27 归档（抖音视频页免登录提取元数据+章节要点；GitHub API + README 9.2K 全文精读核实）
