---
title: 路线图
---

# 路线图

以下只是我们计划在不久的将来实现的一些主要功能的一部分。要更全面的了解计划中的功能和当前的工作，请参阅各个代码库的问题跟踪。例如 [Prometheus](https://github.com/prometheus/prometheus/issues)

### 服务端指标元数据支持 <a id="server-side-metric-metadata-support"></a>

目前，指标数据类型和其它元数据仅在客户端库和暴露格式中使用，而没有在 Prometheus 服务中保留或使用。我们计划将来使用此元数据。第一步是 Prometheus 在内存中聚合此数据，并通过测试 API 端点提供。

### 采用 OpenMetrics <a id="adopt-openmetrics"></a>

OpenMetrics 工作组正在开发数据指标暴露的新标准。我们计划在客户端库和 Prometheus 中支持这种格式。

### 回填时间序列 <a id="backfill-time-series"></a>

回填将允许加载大量之前的数据。这将允许重新进行规则评估和从其它监控系统传输旧数据。

### HTTP 服务端点中的 TLS 和身份验证 <a id="tls-and-authentication-in-http-serving-endpoints"></a>

Prometheus、Alertmanager 和官方 exporter 的 HTTP 服务端点点尚未内置对 TLS 和身份验证的支持。添加此支持将使人们更容易安全地部署 Prometheus 组件，而无需反向代理在外部添加这些功能。

### 生态支持 <a id="support-the-ecosystem"></a>

Prometheus 具有一系列客户端库和 exporter。将来会有更多的语言支持，或更多的暴露数据指标系统。我们将在生态系统创建和扩展它们。

