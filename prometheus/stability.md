---
title: API 稳定性保证
---

# API 稳定性保证

Prometheus 承诺 API 会在主要版本中保持稳定，并努力避免破坏关键功能的更改。某些功能\(例如装饰性的，仍在开发中或依赖于第三方服务的功能\)不在此范围内。

对于 2.x 来说是稳定的：

* 查询语言和数据模型
* 告警和记录规则
* 存储展示格式
* v1 版本的 HTTP API\(由仪表板和UI使用\)
* 配置文件格式\(不包含服务发现远程读/写，请参见下文\)
* 规则/告警文件格式
* 控制台模板语法和语义

对于 2.x 来说是不稳定的：

* 被列为实验性或可能更改的任何功能，包括
  * [`holt_winters` PromQL 函数](https://github.com/prometheus/prometheus/issues/2458)
  * 远程读/写，及远程读端点
  * v2 HTTP 和 GRPC APIs
* 服务发现集成，`static_configs`和`file_sd_configs`除外
* 服务端的软件包 Go API
* Web UI 生成的 HTML
* Prometheus 自身的`/metrics`端点中的数据指标
* 精确的磁盘格式。潜在的更改将由 Prometheus 向前兼容并透明地处理
* 日志格式

只要您不使用任何标记为试验性/不稳定的功能，通常就可以在没有任何调整的情况下执行主要版本的升级，且几乎不会有任何风险。任何重大更改将在发行说明中标记为`CHANGE`。

