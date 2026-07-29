# 从零搭建云原生可观测性平台：Prometheus + Loki + Tempo + OpenTelemetry 实战

> **来源**：白聊科技（埃里克森）  
> **日期**：2026-07-01  
> **原文链接**：https://mp.weixin.qq.com/s/s883tF32btGUOhBM4f1rew  
> **分类**：工具与实用技能

---

## 文章概要

在 Kubernetes 集群上从零搭建完整的可观测性平台，覆盖可观测性三大支柱（Metrics / Logs / Traces），Java 应用通过 OpenTelemetry 自动采集并接入 Prometheus + Loki + Tempo，Grafana 统一可视化。

---

## 架构全景

```
用户访问层 → Java 应用（OTel Java Agent）
                  ↓
         OpenTelemetry Collector
        ↙        ↓        ↘
  Prometheus    Loki      Tempo
  (指标存储)   (日志存储)  (链路存储)
        ↖        ↓        ↗
            Grafana
        (统一可视化大盘)
```

---

## 核心组件

| 组件 | 角色 | 选型理由 |
|:---|:---|:---|
| **Prometheus** | 指标存储 | kube-prometheus-stack 全家桶（含 Alertmanager / Grafana / Node Exporter） |
| **Loki** | 日志存储 | 比 ELK 轻量、与 Grafana 原生集成、不全文索引日志内容 |
| **Tempo** | 链路存储 | 低成本对象存储、原生 OTLP 兼容、Trace→Logs/Metrics 跳转 |
| **OpenTelemetry** | 数据采集 | 统一协议（OTLP）、单 Agent 采集三大信号 |
| **Grafana** | 可视化 | 统一大盘、Trace to Logs 联动 |

---

## 实施步骤

### 1. 环境基础
- Kubernetes v1.33.6 + Helm 3.x
- 已部署 kube-prometheus-stack（Prometheus Operator 全家桶）

### 2. 安装 Tempo（链路追踪存储）
- Helm chart：tempo-1.24.4.tgz
- Grafana/tempo:2.9.0
- 对象存储（MinIO/S3/GCS），低成本方案

### 3. 安装 Loki（日志存储）
- Helm chart：loki-17.4.10.tgz
- 中小集群用 **SingleBinary 单节点模式**
- 本地文件系统存储（10Gi PV），生产可用对象存储
- Promtail 自动采集节点日志

### 4. Java 应用接入 OpenTelemetry
- 添加 `opentelemetry-javaagent.jar` 作为 javaagent
- 日志格式改为 JSON 结构化，携带 `trace_id` / `span_id`
- 环境变量配置：`OTEL_SERVICE_NAME`、`OTEL_EXPORTER_OTLP_ENDPOINT` 等
- 支持自动采集 HTTP、数据库、消息队列等埋点

### 5. 部署 OpenTelemetry Collector
- 通过 CR `OpenTelemetryCollector` 配置
- **Pipelines 配置**：
  - Traces → otlp/tempo
  - Logs → otlphttp/loki
  - Metrics → prometheusremotewrite
- Batch processor：1s 超时 / 1024 条批量

### 6. Grafana 可视化
- 添加 Loki 数据源 → 按 `service_name=java_demo` 查询日志
- 添加 Tempo 数据源 → 按服务搜索 Trace，查看完整调用链
- Trace to Logs / Metrics 联动排查

---

## 关键配置要点

- **日志 JSON 化**：logback-spring.xml 输出 `{timestamp, thread, trace_id, span_id, level, logger, message}`
- **OTel Collector 网络**：配置 dnsConfig 确保集群内服务发现
- **健康检查**：Java 应用配置 livenessProbe / readinessProbe
- **资源限制**：Collector 256Mi/500m，Java 应用 512Mi/1000m

---

## 关键词

`可观测性` `Prometheus` `Loki` `Tempo` `OpenTelemetry` `Grafana` `Kubernetes` `Java Agent` `OTLP` `Metrics` `Logs` `Traces` `云原生`
