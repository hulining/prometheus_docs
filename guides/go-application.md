---
title: 实现一个 Go 应用
---

# 实现一个 Go 应用

Prometheus 具有官方 [Go 客户端库](https://github.com/prometheus/client\_golang) 可用于实现 Go 应用程序。在本指南中，我们将创建一个简单的 Go 应用程序，该应用程序通过 HTTP 公开 Prometheus 指标。

NOTE: 有关全面的 AP I文档，请参见 Prometheus 的各种 Go 库的 [GoDoc](https://godoc.org/github.com/prometheus/client\_golang) 文档。

## 安装 <a href="#installation" id="installation"></a>

您可以使用 [`go get`](https://golang.org/doc/articles/go\_command.html) 安装指南所需的 `prometheus`, `promauto` 和 `promhttp` 库:

```bash
go get github.com/prometheus/client_golang/prometheus
go get github.com/prometheus/client_golang/prometheus/promauto
go get github.com/prometheus/client_golang/prometheus/promhttp
```

## 如何公开 <a href="#how-go-exposition-works" id="how-go-exposition-works"></a>

要在 Go 应用程序中公开 Prometheus 指标,您需要提供 `/metrics` HTTP 端点。您可以使用 [`prometheus/promhttp`](https://godoc.org/github.com/prometheus/client\_golang/prometheus/promhttp) 库的 HTTP [`Handler`](https://godoc.org/github.com/prometheus/client\_golang/prometheus/promhttp#Handler) 作为处理函数。

例如，这个最小的应用程序将通过 `http://localhost:2112/metrics` 公开 Go 应用程序的默认指标:

```go
package main

import (
        "net/http"

        "github.com/prometheus/client_golang/prometheus/promhttp"
)

func main() {
        http.Handle("/metrics", promhttp.Handler())
        http.ListenAndServe(":2112", nil)
}
```

运行该程序:

```bash
go run main.go
```

获取数据指标:

```bash
curl http://localhost:2112/metrics
```

## 添加您自己的数据指标 <a href="#adding-your-own-metrics" id="adding-your-own-metrics"></a>

[上面](broken-reference)的应用程序仅公开默认的 Go 指标。您还可以注册自定义应用程序指定指标。如下示例应用程序公开了 `myapp_processed_ops_total` [计数器](../concepts/metric\_types.md#counter)，该计数器对到目前为止已处理的操作数量进行计数。每2秒，计数器增加1。

```go
package main

import (
        "net/http"
        "time"

        "github.com/prometheus/client_golang/prometheus"
        "github.com/prometheus/client_golang/prometheus/promauto"
        "github.com/prometheus/client_golang/prometheus/promhttp"
)

func recordMetrics() {
        go func() {
                for {
                        opsProcessed.Inc()
                        time.Sleep(2 * time.Second)
                }
        }()
}

var (
        opsProcessed = promauto.NewCounter(prometheus.CounterOpts{
                Name: "myapp_processed_ops_total",
                Help: "The total number of processed events",
        })
)

func main() {
        recordMetrics()

        http.Handle("/metrics", promhttp.Handler())
        http.ListenAndServe(":2112", nil)
}
```

运行该程序:

```bash
go run main.go
```

获取数据指标:

```bash
curl http://localhost:2112/metrics
```

在指标输出中，您将看到 `myapp_processed_ops_total` 计数器的帮助文本，类型信息和当前值：

```
# HELP myapp_processed_ops_total The total number of processed events
# TYPE myapp_processed_ops_total counter
myapp_processed_ops_total 5
```

您可以[配置](../prometheus/configuration/configuration.md#scrape\_config)在本地运行的 Prometheus 实例，从应用程序中获取指标。这是一个 `prometheus.yml` 配置示例：

```yaml
scrape_configs:
- job_name: myapp
  scrape_interval: 10s
  static_configs:
  - targets:
    - localhost:2112
```

## 其它 Go 客户端功能 <a href="#other-go-client-features" id="other-go-client-features"></a>

在本指南中，我们仅介绍了 Prometheus Go 客户端库中的一小部分功能。您还可以公开其他度量标准类型，例如 [gauges](https://godoc.org/github.com/prometheus/client\_golang/prometheus#Gauge) 和 [histograms](https://godoc.org/github.com/prometheus/client\_golang/prometheus#Histogram)，[非全局 registries](https://godoc.org/github.com/prometheus/client\_golang/prometheus#Registry)，用于[推送数据指标](https://godoc.org/github.com/prometheus/client\_golang/prometheus/push)到 Prometheus [PushGateways](../instrumenting/pushing.md)，连接 Prometheus 和 [Graphite](https://godoc.org/github.com/prometheus/client\_golang/prometheus/graphite) 的函数等。

## Summary

在本指南中，您创建了两个示例 Go 应用程序，这些示例向 Prometheus 公开了指标 --- 一个仅公开了默认Go指标，另一个还公开了自定义 Prometheus 计数器 --- 并配置了 Prometheus 实例从这些应用程序中获取指标。
