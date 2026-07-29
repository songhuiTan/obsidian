---
title: "开源，开源，推荐一个基于AI+Agent+MCP+LangChain无人机租赁系统，带文档、源码、详细教程"
source: "程序员小孟"
source_url: "https://mp.weixin.qq.com/s/hEjEt01vb8_v--Y9nvxtBA"
date: "2026-07-17"
tags: [AI, Agent, MCP, LangChain, 开源, Java, Spring Boot, Vue, Spring AI, 无人机, 租赁系统]
---

## 概述

程序员小孟介绍了一套面向 B2C 场景的 AI 无人机租赁系统「翱翔 AI 智能体无人机租赁系统」，已开源并提供详细文档、源码和视频教程。系统围绕"选机、资质、下单、支付、发货、收货、退租、故障报修、维保、评价"形成业务闭环，并嵌入 AI 咨询顾问贯穿全流程。后端已从 Java 8/Spring Boot 2.7 升级至 Java 21/Spring Boot 3.5.13，补齐了 Spring AI Tool Calling、MCP、AI Memory 以及 Dify/Coze 接线说明。

## 核心功能：业务闭环

1. 用户浏览设备列表，按品牌、类型、价格、库存筛选可租机型
2. 用户提交飞行资质，管理员审核决定是否允许下单
3. 用户创建租赁订单，系统计算租赁天数、租金、押金和订单状态
4. 用户完成模拟支付后，管理员发货，用户确认收货进入租赁中状态
5. 租赁结束后用户申请归还，管理员确认归还或退款
6. 设备异常时用户提交故障上报，管理员审核并创建维保工单
7. 订单完成后用户评价，管理员可回复、屏蔽或删除评价
8. AI 咨询顾问贯穿浏览、下单、资质、订单和故障流程，提供上下文感知的辅助决策

## 技术栈

### 后端

| 技术 | 版本 | 用途 |
|------|------|------|
| Java | 21 | 运行时与编译目标 |
| Spring Boot | 3.5.13 | Web 框架、配置管理 |
| Spring MVC | — | REST API、拦截器、文件映射 |
| MyBatis-Plus | 3.5.16 | ORM、分页、逻辑删除 |
| MySQL Driver | — | MySQL 8 连接 |
| Druid | 1.2.28 | 数据库连接池 |
| JJWT | 0.12.6 | JWT 生成与校验 |
| springdoc-openapi | 2.8.17 | Swagger UI |
| Spring AI | 1.1.5 | OpenAI-compatible ChatClient、Tool Calling、MCP WebMVC Server |
| Hutool | 5.8.22 | 通用工具类 |
| Lombok | — | 样板代码简化 |

### 前端

| 技术 | 版本 | 用途 |
|------|------|------|
| Vue | 3.4.21 | 用户端和管理端 UI |
| Vite | 5.2.x | 开发服务器与构建 |
| Element Plus | 2.6.1 | 表单、表格、弹窗 |
| Pinia | 2.1.7 | 登录态与全局状态管理 |
| Vue Router | 4.3.0 | 路由和权限守卫 |
| Axios | 1.6.8 | HTTP 请求封装 |
| ECharts | 6.0.0 | 管理端统计图表 |
| Sass | 1.72.0 | 样式组织 |

### AI 与外部集成

| 组件 | 形态 | 用途 |
|------|------|------|
| Spring AI ChatClient | Java 主链路 | 连接 OpenAI-compatible 云模型并触发 Tool Calling |
| OpenAI-compatible Provider | 可配置 | 可接 OpenAI、DeepSeek、通义千问、百炼、火山方舟、豆包、智谱、硅基流动等 |
| RAGFlow | 可选/降级链路 | 知识库问答、引用来源 |
| MCP WebMVC Server | 可选 | 对外暴露业务工具，供 MCP Client 调用 |
| HTTP Tool API | /api/ai/tools | 供 Dify、Coze 等工作流平台调用 |
| AI Memory | MySQL 持久化 | 记录用户预算、用途、经验等级、品牌偏好等结构化记忆 |
| AI Trace / Tool Log | MySQL 持久化 | 记录 traceId、模型、降级状态、工具调用耗时和结果 |
| FastAPI + LangChain/LangGraph | ai-agent-service/ 可选 | 独立 Agent 编排服务（教学/演示用） |

## 系统架构层次

| 层级 | 组件 | 职责 |
|------|------|------|
| 客户端层 | Vue 3、Vite、Element Plus、Pinia、Axios、ECharts | 用户端、管理端、AI 咨询浮窗 |
| 接口层 | Spring MVC Controller、JWT Interceptor、Admin Interceptor、MCP Interceptor | REST API、登录鉴权、外部 AI 工具鉴权 |
| 业务层 | Service/ServiceImpl、AI Tool、AI Memory、Trace Recorder | 业务与 AI 业务编排 |
| 数据访问层 | MyBatis-Plus、Mapper、Entity | 业务表与 AI 表的 CRUD |
| AI 编排层 | Spring AI ChatClient、OpenAI-compatible Provider、RAGFlow、Tool Calling、MCP、LangChain Client | 云模型优先、工具调用、RAGFlow 降级 |
| 数据存储层 | MySQL 8、uploads 本地文件、RAGFlow 知识库 | 业务数据、文件、知识库 |

## 视频教程

B 站视频教程：https://www.bilibili.com/video/BV1eQNe6PEMa/

## 源码获取

项目发布平台：https://www.pdxmw.com
源码地址：https://www.pdxmw.com/freeProject/80

## 关键洞察

- 这是一个**全栈 AI 业务系统**，不是玩具 demo——从用户选机到故障维保的完整租赁闭环都实现了，AI 嵌入在真实业务上下文中（不是独立聊天框）。
- **技术栈组合非常丰富**：Spring Boot 3.5 + Spring AI 1.1 + MCP + LangChain/LangGraph（可选）+ RAGFlow + Dify/Coze 接入，适合作为 AI 项目作品展示给面试官。
- 支持多种 AI 模型供应商的兼容接口（DeepSeek、通义千问、火山方舟等），不锁定单一厂商。
- 源码通过 pdxmw.com 发布（非 GitHub），可能需要注册平台账号才能获取。

## 战略分析

**与 Hermes Agent 体系的对照：**

该项目与 Hermes Agent 的差异大于相似度。Hermes 是 Agent 运行框架（CLI + 工具 + 网关），而本项目是一个垂直业务系统（租赁平台 + AI 客服）。两者在架构上互补而非冲突。

**差距与借鉴点：**

1. **MCP Server 实践**：项目通过 Spring AI 的 MCP WebMVC Server 把业务工具暴露为 MCP 协议——这个模式值得参考。如果 Hermes 需要对接 Java 业务系统的工具能力，这种 Spring MCP Server 模式是一个标准方案。
2. **AI Memory 持久化**：项目用 MySQL 记录用户预算、偏好、经验等级等结构化记忆，供 AI 做个性化推荐。Hermes 的 TencentDB Memory 也是结构化记忆系统，但面向 Agent 对话历史，而非业务实体。两种记忆模式面向不同场景，没有直接替代关系。
3. **Tool Calling 链路**：Spring AI 的 Tool Calling + MCP + Dify/Coze 三方接线方案展示了在 Java 生态中如何编排 AI 工具调用——这个参考价值主要在 Spring Boot 开发者的工具链选择，对 Hermes（Python/TS 生态）的参考有限。

**整合可能性：** 低。这是一个独立的业务项目，不是通用框架。但如果未来需要 Agent 调用 Java 业务系统的能力，建议参考其 MCP Server 接线方式——通过 Spring AI 的 MCP 模块暴露 REST 工具接口。

## 归档日志

- 2026-07-17 归档
