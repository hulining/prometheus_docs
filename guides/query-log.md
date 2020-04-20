---
title: Query Log
sort_rank: 1
---

# 使用 Prometheus 查询日志

从 2.16.0 开始，Prometheus 可以将引擎运行的所有查询记录到一个日志文件中。本指南演示了如何使用该日志文件及其包含的字段，并提供了有关如何操作日志文件的高级技巧。

## 启用查询日志 <a id="enable-the-query-log"></a>

可以在运行时切换查询日志。当您要研究 Prometheus 实例变慢或高负载时，可以将其激活。

要启用或禁用查询日志，需要执行两个步骤:

1. 修改配置以添加或删除查询日志配置
2. 重新加载 Prometheus 服务配置.

### 将所有查询记录到文件 <a id="logging-all-the-queries-to-a-file"></a>

此示例演示如何将所有查询记录到名为 `/prometheus/query.log` 的文件中。我们假定 `/prometheus` 是数据目录，且 Prometheus 对此目录具有写权限 This example demonstrates how to log all the queries to a file called `/prometheus/query.log`. We will assume that `/prometheus` is the data directory and that Prometheus has write access to it.

首先，修改 `prometheus.yml` 配置文件

```yaml
global:
  scrape_interval:     15s
  evaluation_interval: 15s
  query_log_file: /prometheus/query.log
scrape_configs:
- job_name: 'prometheus'
  static_configs:
  - targets: ['localhost:9090']
```

然后, [重新加载](https://prometheus.io/docs/guides/prometheus/latest/management_api/#reload) Prometheus 配置:

```text
$ curl -X POST http://127.0.0.1:9090/-/reload
```

或者，如果 Prometheus 没有使用 `--web.enable-lifecycle` 标志启动，且您不在 Windows 上运行，则可以通过将 `SIGHUP` 发送给 Prometheus 进程来触发重新加载。

`/prometheus/query.log` 文件现在应该存在，所有查询都将记录到该文件中。

要禁用查询日志，从配置中删除 `query_log_file` 后，重复该操作

## 验证查询日志是否启用 <a id="verifying-if-the-query-log-is-enabled"></a>

Prometheus 公开了指示查询日志是否启用并正常工作的数据指标:

```text
# HELP prometheus_engine_query_log_enabled State of the query log.
# TYPE prometheus_engine_query_log_enabled gauge
prometheus_engine_query_log_enabled 0
# HELP prometheus_engine_query_log_failures_total The number of query log failures.
# TYPE prometheus_engine_query_log_failures_total counter
prometheus_engine_query_log_failures_total 0
```

如果查询日志已启用，则如上所示的第一个数据指标 `prometheus_engine_query_log_enabled` 将设置为 1，否则设置为0。第二个数据指标 `prometheus_engine_query_log_failures_total` 表示无法记录的查询数。

## 查询日志的格式 <a id="format-of-the-query-log"></a>

查询日志是 JSON 格式的日志。这是查询中字段的概述：

```text
{
    "params": {
        "end": "2020-02-08T14:59:50.368Z",
        "query": "up == 0",
        "start": "2020-02-08T13:59:50.368Z",
        "step": 5
    },
    "stats": {
        "timings": {
            "evalTotalTime": 0.000447452,
            "execQueueTime": 7.599e-06,
            "execTotalTime": 0.000461232,
            "innerEvalTime": 0.000427033,
            "queryPreparationTime": 1.4177e-05,
            "resultSortTime": 6.48e-07
        }
    },
    "ts": "2020-02-08T14:59:50.387Z"
}
```

* `params`: 封装的查询信息，包含查询相关参数: 开始和结束时间，步长和实际的查询语句
* `stats`: 封装的状态信息.此示例中，包含内部引擎计时器
* `ts`: 查询结束的时间戳

此外，根据发送的请求，JSON 行中还可能包含其他字段。

### API 查询和控制台 <a id="api-queries-and-consoles"></a>

HTTP 请求包含客户端 IP，方法和路径:

```text
{
    "httpRequest": {
        "clientIP": "127.0.0.1",
        "method": "GET",
        "path": "/api/v1/query_range"
    }
}
```

`path` 将包含 Web 前缀\(如果已设置\)，并且可以指向控制台。

`clientIP` 是网络 IP 地址，并且不考虑诸如 `X-Forwarded-For` 之类的请求头。如果您想要记录在代理之后的原始请求者，则需要在代理本身中进行记录。

### 记录规则和告警 <a id="recording-rules-and-alerts"></a>

记录规则和告警包含 `ruleGroup` 元素，该元素包含文件的路径和组的名称：

```text
{
    "ruleGroup": {
        "file": "rules.yml",
        "name": "partners"
    }
}
```

## 查询日志滚动 <a id="rotating-the-query-log"></a>

Prometheus 不会自己滚动查询日志。您可以使用外部工具来执行此操作。

logrotate 是工具之一。大多数 Linux 发行版都默认可以使用该功能

如下是您可以添加为 `/etc/logrotate.d/prometheus` 的文件示例:

```text
/prometheus/query.log {
    daily
    rotate 7
    compress
    delaycompress
    postrotate
        killall -HUP prometheus
    endscript
}
```

这将每天滚动一次日志文件，并保留一个星期的历史记录。

