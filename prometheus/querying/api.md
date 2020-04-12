---
title: HTTP API
---

# HTTP API

在 Prometheus 服务的`/api/v1`下可以访问当前稳定的 HTTP API。任何不间断的添加都将添加到该端点下。

## 格式概述 <a id="format-overview"></a>

API 响应格式为 JSON。每个成功的 API 请求都会返回 2xx 状态码。

到达 API 处理程序的无效请求会返回 JSON 错误对象和以下 HTTP 状态码之一：

* `400 Bad Request`: 参数丢失或不正确
* `422 Unprocessable Entity`: 表达式无法被执行\([RFC4918](https://tools.ietf.org/html/rfc4918#page-78)\)
* `503 Service Unavailable`: 查询超时或中止

对于到达 API 端点之前发生的错误，可能返回其它非`2xx`状态码。

如果存在不会阻止请求执行的错误，则可能会返回一系列警告。成功收集的所有数据都将在 data 字段中返回。

JSON 响应格式封装如下：

```text
{
  "status": "success" | "error",
  "data": <data>,

  // 仅在 status 为 "error" 时设置。data 字段可能仍包含其他数据。
  "errorType": "<string>",
  "error": "<string>",

  // 仅在执行请求时有警告。data 字段可能仍包含其他数据。
  "warnings": ["<string>"]
}
```

输入时间戳可以以 [RFC3339](https://www.ietf.org/rfc/rfc3339.txt) 格式提供，也可以以 Unix 时间戳\(以秒为单位\)提供，可选的小数位用于亚秒级精度。输出时间戳始终以秒为单位表示为 Unix 时间戳。

可以重复的查询参数的名称以`[]`结尾。

`<series_selector>`占位符指的是 Prometheus [时间序列选择器](basics.md#time-series-selectors)，例如`http_requests_total`或`http_requests_total{method=~"(GET|POST)"}`，且需要进行 URL 编码。

`<duration>` 占位符是指形式为`[0-9]+[smhdwy]`的 Prometheus 持续时间字符串。例如，5m 表示持续时间为 5 分钟。

`<bool>`占位符引用布尔值\(字符串`true`和`false`\)。

## 表达式查询 <a id="expression-queries"></a>

查询语言表达式可以获取单个瞬间或一段时间的值。以下各节介绍了每种类型的表达式的查询的 API 端点。

### 瞬时查询 <a id="instant-queries"></a>

以下端点可以在单个时间点计算即时查询：

```text
GET /api/v1/query
POST /api/v1/query
```

URL 查询参数：

* `query=<string>`: Prometheus 表达式查询字符串
* `time=<rfc3339 | unix_timestamp>`: 评估时间戳。可选的
* `timeout=<duration>`: 评估的超时时间。可选的。默认值为`-query.timeou`标志设定值

如果省略`time`参数，则使用当前服务器时间。

您可以使用`POST`方法和`Content-Type: application/x-www-form-urlencoded`请求头直接在请求正文中对这些参数进行 URL 编码。当指定一个可能违反服务器端 URL 字符限制的大型查询时，此功能很有用。

查询结果的`data`部分具有如下格式：

```text
{
  "resultType": "matrix" | "vector" | "scalar" | "string",
  "result": <value>
}
```

`<value>`表示查询结果数据，其格式取决于`resultType`。查看[表达式查询结果格式](api.md#expression-query-result-formats)

以下示例在`2015-07-01T20:10:51.781Z`时间计算表达式`up`：

```text
$ curl 'http://localhost:9090/api/v1/query?query=up&time=2015-07-01T20:10:51.781Z'
{
   "status" : "success",
   "data" : {
      "resultType" : "vector",
      "result" : [
         {
            "metric" : {
               "__name__" : "up",
               "job" : "prometheus",
               "instance" : "localhost:9090"
            },
            "value": [ 1435781451.781, "1" ]
         },
         {
            "metric" : {
               "__name__" : "up",
               "job" : "node",
               "instance" : "localhost:9100"
            },
            "value" : [ 1435781451.781, "0" ]
         }
      ]
   }
}
```

### 范围查询 <a id="range-queries"></a>

以下端点可以在一段时间内评估表达式查询：

```text
GET /api/v1/query_range
POST /api/v1/query_range
```

URL 查询参数：

* `query=<string>`: Prometheus 表达式查询字符串
* `start=<rfc3339 | unix_timestamp>`: 开始时间戳。
* `end=<rfc3339 | unix_timestamp>`: 结束时间戳
* `step=<duration | float>`: 以持续时间格式或浮点秒数为分辨率步长进行查询
* `timeout=<duration>`: 评估的超时时间。可选的。默认值为`-query.timeou`标志设定值

您可以使用`POST`方法和`Content-Type: application/x-www-form-urlencoded`请求头直接在请求正文中对这些参数进行 URL 编码。当指定一个可能违反服务器端 URL 字符限制的大型查询时，此功能很有用。

查询结果的`data`部分具有如下格式：

```text
{
  "resultType": "matrix",
  "result": <value>
}
```

有关`<value>`占位符的格式，请参见[范围向量结果格式](api.md#range-vectors)。

下面的示例在 30 秒范围内以 15 秒的查询分辨率对表达式`up`进行求值：

```text
$ curl 'http://localhost:9090/api/v1/query_range?query=up&start=2015-07-01T20:10:30.781Z&end=2015-07-01T20:11:00.781Z&step=15s'
{
   "status" : "success",
   "data" : {
      "resultType" : "matrix",
      "result" : [
         {
            "metric" : {
               "__name__" : "up",
               "job" : "prometheus",
               "instance" : "localhost:9090"
            },
            "values" : [
               [ 1435781430.781, "1" ],
               [ 1435781445.781, "1" ],
               [ 1435781460.781, "1" ]
            ]
         },
         {
            "metric" : {
               "__name__" : "up",
               "job" : "node",
               "instance" : "localhost:9091"
            },
            "values" : [
               [ 1435781430.781, "0" ],
               [ 1435781445.781, "0" ],
               [ 1435781460.781, "1" ]
            ]
         }
      ]
   }
}
```

## 查询元数据 <a id="querying-metadata"></a>

### 通过标签匹配器查找序列 <a id="finding-series-by-label-matchers"></a>

以下端点返回与某个标签集匹配的时间序列列表。

```text
GET /api/v1/series
POST /api/v1/series
```

URL 查询参数

* `match[]=<series_selector>`: 重复的序列选择器参数，用于选择要返回的序列。必须至少提供一个`match []`参数。
* `start=<rfc3339 | unix_timestamp>`: 开始时间戳。
* `end=<rfc3339 | unix_timestamp>`: 结束时间戳。

您可以使用`POST`方法和`Content-Type: application/x-www-form-urlencoded`请求头直接在请求正文中对这些参数进行 URL 编码。当指定一个可能违反服务器端 URL 字符限制的大型查询时，此功能很有用。

查询结果的`data`部分有一个对象列表组成，这些对象包含标识每个序列的标签名称/值对。

以下示例返回与`up`或`process_start_time_seconds{job="prometheus"}`匹配的所有序列：

```text
$ curl -g 'http://localhost:9090/api/v1/series?' --data-urlencode 'match[]=up' --data-urlencode 'match[]=process_start_time_seconds{job="prometheus"}'
{
   "status" : "success",
   "data" : [
      {
         "__name__" : "up",
         "job" : "prometheus",
         "instance" : "localhost:9090"
      },
      {
         "__name__" : "up",
         "job" : "node",
         "instance" : "localhost:9091"
      },
      {
         "__name__" : "process_start_time_seconds",
         "job" : "prometheus",
         "instance" : "localhost:9090"
      }
   ]
}
```

### 获取标签名称 <a id="getting-label-names"></a>

下面的端点返回标签名称的列表

```text
GET /api/v1/labels
POST /api/v1/labels
```

JSON 响应的`data`部分是字符串标签名称的列表

以下是一个示例：

```text
$ curl 'localhost:9090/api/v1/labels'
{
    "status": "success",
    "data": [
        "__name__",
        "call",
        "code",
        "config",
        "dialer_name",
        "endpoint",
        "event",
        "goversion",
        "handler",
        "instance",
        "interval",
        "job",
        "le",
        "listener_name",
        "name",
        "quantile",
        "reason",
        "role",
        "scrape_job",
        "slice",
        "version"
    ]
}
```

### 查询标签值 <a id="querying-label-values"></a>

以下端点返回给定标签名称的标签值的列表

```text
GET /api/v1/label/<label_name>/values
```

JSON 响应的`data`部分是字符串标签值的列表

以下是查询`job`标签的所有标签值的示例：

```text
$ curl http://localhost:9090/api/v1/label/job/values
{
   "status" : "success",
   "data" : [
      "node",
      "prometheus"
   ]
}
```

## 表达查询结果格式 <a id="expression-query-result-formats"></a>

表达式查询可能在`result`属性的`data`部分返回以下响应值。`<sample_value>`占位符是数字样本值。JSON不支持`NaN`, `Inf`和`-Inf`等特殊的浮点值，因此`<sample_value>`将作为带引号的 JSON 字符串而不是原始数字进行传输。

### 范围向量 <a id="range-vectors"></a>

范围向量结果作为`matrix`结果类型返回。相应的`result`属性具有如下格式：

```text
[
  {
    "metric": { "<label_name>": "<label_value>", ... },
    "values": [ [ <unix_time>, "<sample_value>" ], ... ]
  },
  ...
]
```

### 即时向量 <a id="instant-vectors"></a>

即时向量结果作为`vector`结果类型返回。相应的`result`属性具有如下格式：

```text
[
  {
    "metric": { "<label_name>": "<label_value>", ... },
    "value": [ <unix_time>, "<sample_value>" ]
  },
  ...
]
```

### 标量 <a id="scalars"></a>

标量结果作为`scalar`结果类型返回。相应的`result`属性具有如下格式：

```text
[ <unix_time>, "<scalar_value>" ]
```

### 字符串 <a id="strings"></a>

字符串结果作为`string`结果类型返回。相应的`result`属性具有如下格式：

```yaml
[ <unix_time>, "<string_value>" ]
```

## 目标 <a id="targets"></a>

以下端点返回 Prometheus 发现目标的当前状态概述：

```text
GET /api/v1/targets
```

默认情况下，活动或已删除的目标都是响应的一部分。`labels`表示重新标记发生后的标签集。`discoveredLabels`表示在重新标记发生之前在服务发现期间检索到的未修改标签。

```text
$ curl http://localhost:9090/api/v1/targets
{
  "status": "success",
  "data": {
    "activeTargets": [
      {
        "discoveredLabels": {
          "__address__": "127.0.0.1:9090",
          "__metrics_path__": "/metrics",
          "__scheme__": "http",
          "job": "prometheus"
        },
        "labels": {
          "instance": "127.0.0.1:9090",
          "job": "prometheus"
        },
        "scrapePool": "prometheus",
        "scrapeUrl": "http://127.0.0.1:9090/metrics",
        "lastError": "",
        "lastScrape": "2017-01-17T15:07:44.723715405+01:00",
        "lastScrapeDuration": 0.050688943,
        "health": "up"
      }
    ],
    "droppedTargets": [
      {
        "discoveredLabels": {
          "__address__": "127.0.0.1:9100",
          "__metrics_path__": "/metrics",
          "__scheme__": "http",
          "job": "node"
        },
      }
    ]
  }
}
```

`state`查询参数允许调用者按活动或已删除的目标进行过滤\(例如：`state=active`, `state=dropped`, `state=any`\)。请注意，对于已滤除的目标，仍然返回空数组。其它值将被忽略。

```text
$ curl 'http://localhost:9090/api/v1/targets?state=active'
{
  "status": "success",
  "data": {
    "activeTargets": [
      {
        "discoveredLabels": {
          "__address__": "127.0.0.1:9090",
          "__metrics_path__": "/metrics",
          "__scheme__": "http",
          "job": "prometheus"
        },
        "labels": {
          "instance": "127.0.0.1:9090",
          "job": "prometheus"
        },
        "scrapePool": "prometheus",
        "scrapeUrl": "http://127.0.0.1:9090/metrics",
        "lastError": "",
        "lastScrape": "2017-01-17T15:07:44.723715405+01:00",
        "lastScrapeDuration": 50688943,
        "health": "up"
      }
    ],
    "droppedTargets": []
  }
}
```

## 规则 <a id="rules"></a>

`/rules` API 端点返回当前已加载的告警和记录规则的列表。此外，它还返回由 Prometheus 实例触发的当前活动的告警的每个警报规则。

由于`/rules` 端点相当新，它没有与总体 API v1 有相同的稳定性保证。

```text
GET /api/v1/rules
```

URL 查询参数：

* `type=alert|record`: 仅返回告警规则\(`type=alert`\)或记录规则\(`type=record`\)。如果该参数不存在或为空，则不执行任何过滤。

```text
$ curl http://localhost:9090/api/v1/rules

{
    "data": {
        "groups": [
            {
                "rules": [
                    {
                        "alerts": [
                            {
                                "activeAt": "2018-07-04T20:27:12.60602144+02:00",
                                "annotations": {
                                    "summary": "High request latency"
                                },
                                "labels": {
                                    "alertname": "HighRequestLatency",
                                    "severity": "page"
                                },
                                "state": "firing",
                                "value": "1e+00"
                            }
                        ],
                        "annotations": {
                            "summary": "High request latency"
                        },
                        "duration": 600,
                        "health": "ok",
                        "labels": {
                            "severity": "page"
                        },
                        "name": "HighRequestLatency",
                        "query": "job:request_latency_seconds:mean5m{job=\"myjob\"} > 0.5",
                        "type": "alerting"
                    },
                    {
                        "health": "ok",
                        "name": "job:http_inprogress_requests:sum",
                        "query": "sum(http_inprogress_requests) by (job)",
                        "type": "recording"
                    }
                ],
                "file": "/rules.yaml",
                "interval": 60,
                "name": "example"
            }
        ]
    },
    "status": "success"
}
```

## 告警 <a id="alerts"></a>

`/alerts`端点返回所有活动告警的列表

由于`/rules` 端点相当新，它没有与总体 API v1 有相同的稳定性保证。

```text
GET /api/v1/rules
```

```text
$ curl http://localhost:9090/api/v1/alerts

{
    "data": {
        "alerts": [
            {
                "activeAt": "2018-07-04T20:27:12.60602144+02:00",
                "annotations": {},
                "labels": {
                    "alertname": "my-alert"
                },
                "state": "firing",
                "value": "1e+00"
            }
        ]
    },
    "status": "success"
}
```

## 查询目标元数据 <a id="querying-target-metadata"></a>

以下端点返回有关当前从目标中采集数据指标的元数据。这是**实验性的**，将来可能会改变。

```text
GET /api/v1/targets/metadata
```

URL 查询参数：

* `match_target=<label_selectors>`: 通过标签集匹配目标的标签选择器。如果保留为空，则选择所有目标。
* `metric=<string>`: 要为其检索的元数据的度量名称。如果保留为空，则选择所有目标。
* `limit=<number>`: 匹配的最大目标数量

查询结果的`data`部分由包含数据指标元数据和目标标签集的对象列表组成。

以下示例从前两个目标中返回带有`job="prometheus"`标签的`go_goroutines`数据指标的所有元数据条目：

```text
curl -G http://localhost:9091/api/v1/targets/metadata \
    --data-urlencode 'metric=go_goroutines' \
    --data-urlencode 'match_target={job="prometheus"}' \
    --data-urlencode 'limit=2'
{
  "status": "success",
  "data": [
    {
      "target": {
        "instance": "127.0.0.1:9090",
        "job": "prometheus"
      },
      "type": "gauge",
      "help": "Number of goroutines that currently exist.",
      "unit": ""
    },
    {
      "target": {
        "instance": "127.0.0.1:9091",
        "job": "prometheus"
      },
      "type": "gauge",
      "help": "Number of goroutines that currently exist.",
      "unit": ""
    }
  ]
}
```

如下示例返回带有`instance="127.0.0.1:9090"`标签的所有目标的所有指标的元数据：

```text
curl -G http://localhost:9091/api/v1/targets/metadata \
    --data-urlencode 'match_target={instance="127.0.0.1:9090"}'
{
  "status": "success",
  "data": [
    // ...
    {
      "target": {
        "instance": "127.0.0.1:9090",
        "job": "prometheus"
      },
      "metric": "prometheus_treecache_zookeeper_failures_total",
      "type": "counter",
      "help": "The total number of ZooKeeper failures.",
      "unit": ""
    },
    {
      "target": {
        "instance": "127.0.0.1:9090",
        "job": "prometheus"
      },
      "metric": "prometheus_tsdb_reloads_total",
      "type": "counter",
      "help": "Number of times the database reloaded block data from disk.",
      "unit": ""
    },
    // ...
  ]
}
```

## 查询数据指标元数据 <a id="querying-metric-metadata"></a>

它返回有关当前从目标中采集数据指标的元数据。但它不提供任何目标信息。这是**实验性的**，将来可能会改变

```text
GET /api/v1/metadata
```

URL 查询参数：

* `limit=<number>`: 指标的最大返回数量
* `metric=<string>`: 用于过滤元数据的数据指标文件。如果保留为空，则将检索所有指标元数据。

查询结果的`data`部分由键是数据指标名称，值都是唯一元数据对象的列表的对象组成，该数据指标名称在所有目标中都暴露。

以下示例返回两个数据指标。请注意，`http_requests_total`数据指标列表中有多个对象。至少一个目标的`HELP`值与其他目标不匹配

```text
curl -G http://localhost:9090/api/v1/metadata?limit=2

{
  "status": "success",
  "data": {
    "cortex_ring_tokens": [
      {
        "type": "gauge",
        "help": "Number of tokens in the ring",
        "unit": ""
      }
    ],
    "http_requests_total": [
      {
        "type": "counter",
        "help": "Number of HTTP requests",
        "unit": ""
      },
      {
        "type": "counter",
        "help": "Amount of HTTP requests",
        "unit": ""
      }
    ]
  }
}
```

如下示例仅返回指标`http_requests_total`的元数据：

```text
curl -G http://localhost:9090/api/v1/metadata?metric=http_requests_total

{
  "status": "success",
  "data": {
    "http_requests_total": [
      {
        "type": "counter",
        "help": "Number of HTTP requests",
        "unit": ""
      },
      {
        "type": "counter",
        "help": "Amount of HTTP requests",
        "unit": ""
      }
    ]
  }
}
```

## Alertmanagers

以下端点返回发现的 Prometheus alertmanager 当前状态的概述：

```text
GET /api/v1/alertmanagers
```

响应包含状态为活动或已删除的 Alertmanagers。

```text
$ curl http://localhost:9090/api/v1/alertmanagers
{
  "status": "success",
  "data": {
    "activeAlertmanagers": [
      {
        "url": "http://127.0.0.1:9090/api/v1/alerts"
      }
    ],
    "droppedAlertmanagers": [
      {
        "url": "http://127.0.0.1:9093/api/v1/alerts"
      }
    ]
  }
}
```

## 状态 <a id="status"></a>

以下状态端点暴露了当前 Prometheus 的配置。

### 配置 <a id="config"></a>

以下端点返回当前加载的配置文件

```text
GET /api/v1/status/config
```

返回的配置作为转储的 YAML 文件返回。由于 YAML 库的限制，不包括 YAML 注释。

```text
$ curl http://localhost:9090/api/v1/status/config
{
  "status": "success",
  "data": {
    "yaml": "<content of the loaded config file in YAML>",
  }
}
```

### 标志位 <a id="flags"></a>

以下端点返回已配置的 Prometheus 的标志位：

```text
GET /api/v1/status/flags
```

所有值均为`string`结果类型

```text
$ curl http://localhost:9090/api/v1/status/flags
{
  "status": "success",
  "data": {
    "alertmanager.notification-queue-capacity": "10000",
    "alertmanager.timeout": "10s",
    "log.level": "info",
    "query.lookback-delta": "5m",
    "query.max-concurrency": "20",
    ...
  }
}
```

_v2.2 的新功能_

### 运行时信息 <a id="runtime-information"></a>

以下端点返回有关 Prometheus 服务的各种运行时信息属性：

```text
GET /api/v1/status/runtimeinfo
```

返回值具有不同的类型，具体取决与运行时属性的性质。

```text
$ curl http://localhost:9090/api/v1/status/runtimeinfo
{
  "status": "success",
  "data": {
    "startTime": "2019-11-02T17:23:59.301361365+01:00",
    "CWD": "/",
    "reloadConfigSuccess": true,
    "lastConfigTime": "2019-11-02T17:23:59+01:00",
    "chunkCount": 873,
    "timeSeriesCount": 873,
    "corruptionCount": 0,
    "goroutineCount": 48,
    "GOMAXPROCS": 4,
    "GOGC": "",
    "GODEBUG": "",
    "storageRetention": "15d"
  }
}
```

{% hint style="warning" %}
在 Prometheus 版本之间，返回的确切运行时属性可能会更改，恕不另行通知。
{% endhint %}

_v2.14的新功能_

### 构建信息 <a id="build-information"></a>

以下端点返回有关 Prometheus 服务的各种构建信息属性：

```text
GET /api/v1/status/buildinfo
```

所有值都是`string`结果类型

```text
$ curl http://localhost:9090/api/v1/status/buildinfo
{
  "status": "success",
  "data": {
    "version": "2.13.1",
    "revision": "cb7cbad5f9a2823a622aaa668833ca04f50a0ea7",
    "branch": "master",
    "buildUser": "julius@desktop",
    "buildDate": "20191102-16:19:59",
    "goVersion": "go1.13.1"
  }
}
```

{% hint style="warning" %}
在 Prometheus 版本之间，返回的确切运行时属性可能会更改，恕不另行通知。
{% endhint %}

_v2.14的新功能_

### TSDB 状态 <a id="tsdb-stats"></a>

以下端点返回有关 Prometheus TSDB 的各种基数统计信息：

```text
GET /api/v1/status/tsdb
```

* **seriesCountByMetricName**: 这将提供指标名称及其序列计数的列表。
* **labelValueCountByLabelName**: 这将提供标签名称及其值计数的列表。
* **memoryInBytesByLabelName**: 这将提供标签名称和以字节为单位使用的内存的列表。内存使用量是通过将给定标签名称的所有值的长度相加得出的。
* **seriesCountByLabelPair**: 这将提供标签值对及其系列计数的列表。

  ```text
  $ curl http://localhost:9090/api/v1/status/tsdb
  {
  "status": "success",
  "data": {
    "seriesCountByMetricName": [
      {
        "name": "net_conntrack_dialer_conn_failed_total",
        "value": 20
      },
      {
        "name": "prometheus_http_request_duration_seconds_bucket",
        "value": 20
      }
    ],
    "labelValueCountByLabelName": [
      {
        "name": "__name__",
        "value": 211
      },
      {
        "name": "event",
        "value": 3
      }
    ],
    "memoryInBytesByLabelName": [
      {
        "name": "__name__",
        "value": 8266
      },
      {
        "name": "instance",
        "value": 28
      }
    ],
    "seriesCountByLabelValuePair": [
      {
        "name": "job=prometheus",
        "value": 425
      },
      {
        "name": "instance=localhost:9090",
        "value": 425
      }
    ]
  }
  }
  ```

  _v2.15 的新功能_

## TSDB 管理 API <a id="tsdb-admin-apis"></a>

这些 API 为高级用户提供了数据库功能。除非设置了`--web.enable-admin-api`，否则不会启用这些 API。

我们还暴露了一个 gRPC API，其定义可以在[此处](https://github.com/prometheus/prometheus/blob/master/prompb/rpc.proto)找到。 这是实验性的，将来可能会改变。

## 快照 <a id="snapshot"></a>

Snapshot 将所有当前数据的快照创建到 TSDB 数据目录下的`snapshots/<datetime>-<rand>`，并返回该目录作为响应。它可以选择跳过仅存在于起始块中并且尚未压缩到磁盘的快照数据。

```text
POST /api/v1/admin/tsdb/snapshot
PUT /api/v1/admin/tsdb/snapshot
```

URL 查询参数：

* `skip_head=<bool>`: 跳过起始块中存在的数据。可选的。

  ```text
  $ curl -XPOST http://localhost:9090/api/v1/admin/tsdb/snapshot
  {
  "status": "success",
  "data": {
    "name": "20171210T211224Z-2be650b6d019eb54"
  }
  }
  ```

  快照保存在`<data-dir>/snapshots/20171210T211224Z-2be650b6d019eb54`。

_v2.1 的新功能，从 v2.9 开始支持 PUT_

### 删除序列 <a id="delete-series"></a>

DeleteSeries 删除一个时间范围内选定序列的数据。实际数据仍然存在于磁盘上，并在以后的压缩中进行清理，或者可以通过访问 Clean Tombstones 端点进行显式清理。

如果成功，响应状态码为`204`。

```text
POST /api/v1/admin/tsdb/delete_series
PUT /api/v1/admin/tsdb/delete_series
```

URL 查询参数:

* `match[]=<series_selector>`: 重复的标签匹配器参数，用于选择要删除的序列。必须至少提供一个`match[]`参数
* `start=<rfc3339 | unix_timestamp>`: 开始时间戳。可选，默认为所有序列时间范围内的最小时间。
* `end=<rfc3339 | unix_timestamp>`: 结束时间戳。可选，默认为所有序列时间范围内的最大时间。

示例：

```text
$ curl -X POST \
  -g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]=up&match[]=process_start_time_seconds{job="prometheus"}'
```

_v2.1 的新功能，从 v2.9 开始支持 PUT_

### Clean Tombstones

CleanTombstones 从磁盘上删除已逻辑删除的数据，并清理现有的逻辑删除。删除序列后可以使用它来释放空间。

如果成功，响应状态码为`204`。

```text
POST /api/v1/admin/tsdb/clean_tombstones
PUT /api/v1/admin/tsdb/clean_tombstones
```

该动作不带参数或请求体。

```text
$ curl -XPOST http://localhost:9090/api/v1/admin/tsdb/clean_tombstones
```

_v2.1 的新功能，从 v2.9 开始支持 PUT_

