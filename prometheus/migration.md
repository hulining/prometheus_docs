---
title: 向 Prometheus 2.0 迁移的指南
---

# Prometheus 2.0 迁移指南

秉承我们[稳定性承诺](https://prometheus.io/blog/2016/07/18/prometheus-1-0-released/#fine-print)，Prometheus 2.0 版本包含许多向后不兼容的更改。本文档提供了从 Prometheus 1.8 迁移到 Prometheus 2.0 的指南。

## 标志 <a id="flags"></a>

Prometheus 命令行标志的格式已经更改。现在所有标志不再使用单破折号，而是使用双破折号。通用标志\(`--config.file`, `--web.listen-address`和`--web.external-url`\)仍然相同，但除此之外，几乎所有与存储的标志都已删除。

一些值的注意的标志已被删除：

* `-alertmanager.url` 在 Prometheus 2.0 中，用于静态配置 Alertmanager 的 URL 命令行标志已被删除。现在必须通过服务发现来发现 Alertmanager。请参阅 [Alertmanager 服务发现](migration.md#alertmanager-service-discovery)。
* `-log.format` 在 Prometheus 2.0 中，日志只能流式传输到标准输出。
* `-query.staleness-delta`已重命名为`--query.lookback-delta`；Prometheus 2.0 引入了一种新的处理陈旧性的机制，请参见[陈旧性](querying/basics.md#staleness)
* `-storage.local.*` Prometheus 2.0 引入了新的存储引擎，因为与旧引擎有关的标志都已删除。有关新引擎的信息，请参阅[存储](https://prometheus.io/docs/prometheus/latest/migration/#storage)部分。
* `-storage.remote.*` Prometheus 2.0 删除了已经废弃的远程存储标志。如果指定了它们将无法启动。要写入 InfluxDB, Graphite 或 OpenTSDB 远程存储，请使用相关的存储适配器。

## Alertmanager 服务发现 <a id="alertmanager-service-discovery"></a>

Prometheus 1.4 中引入了 Alertmanager 服务发现机制，这使得 Prometheus 可以使用与采集数据相同的机制来动态发现 Alertmanager 副本。在 Prometheus 2.0，静态 Alertmanager 配置的命令行标志已删除，因此命令行标志`./prometheus -alertmanager.url=http://alertmanager:9093/`将与`prometheus.yml`配置文件中改为：

```yaml
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - alertmanager:9093
```

您还可以在 alertmanager 配置中使用所有常规的 Prometheus 服务发现集成组件和标签重新标记过程。该代码片段指示 Prometheus 在`default`名称空间中搜索标签为`name: alertmanager`且具有非空端口的 Kubernetes pods.

```yaml
alerting:
  alertmanagers:
  - kubernetes_sd_configs:
      - role: pod
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
    - source_labels: [__meta_kubernetes_pod_label_name]
      regex: alertmanager
      action: keep
    - source_labels: [__meta_kubernetes_namespace]
      regex: default
      action: keep
    - source_labels: [__meta_kubernetes_pod_container_port_number]
      regex:
      action: drop
```

## 记录规则和告警规则 <a id="recording-rules-and-alerts"></a>

告警规则和记录规则的配置格式已更改为 YAML。旧的格式的记录规则和告警规则示例：

```text
job:request_duration_seconds:histogram_quantile99 =
  histogram_quantile(0.99, sum by (le, job) (rate(request_duration_seconds_bucket[1m])))

ALERT FrontendRequestLatency
  IF job:request_duration_seconds:histogram_quantile99{job="frontend"} > 0.1
  FOR 5m
  ANNOTATIONS {
    summary = "High frontend request latency",
  }
```

将被修改为如下格式：

```yaml
groups:
- name: example.rules
  rules:
  - record: job:request_duration_seconds:histogram_quantile99
    expr: histogram_quantile(0.99, sum(rate(request_duration_seconds_bucket[1m]))
      BY (le, job))
  - alert: FrontendRequestLatency
    expr: job:request_duration_seconds:histogram_quantile99{job="frontend"} > 0.1
    for: 5m
    annotations:
      summary: High frontend request latency
```

为了帮助更改，`promtool`工具提供了一种自动转换规则的方法。给定一个`.rules`文件，它将输出新格式的`.rules.yml`。如：

```bash
$ promtool update rules example.rules
```

请注意，您需要使用 2.0 版而非 1.8 版的 promtool。

## 存储 <a id="storage"></a>

Prometheus 2.0 中的数据格式已完全更改，并与 1.8 版本不向后兼容。为了保留对历史监控数据的访问权限，我们建议您运行一个没有采集数据的 1.8.1 版本以上的 Prometheus 实例与 2.0 版本的 Prometheus 实例并行运行，并让新服务通过远程读取协议从旧服务读取现有数据。

您的 1.8 版本的 Prometheus 实例应使用以下标志和仅包含`external_labels`设置\(如果有\)的配置文件启动：

```bash
$ ./prometheus-1.8.1.linux-amd64/prometheus -web.listen-address ":9094" -config.file old.yml
```

然后可以使用以下标志\(在同一台计算机上\)启动 Prometheus 2.0 实例：

```bash
$ ./prometheus-2.0.0.linux-amd64/prometheus --config.file prometheus.yml
```

除了完整的已存在的配置外，`prometheus.yml`还包含以下部分：

```yaml
remote_read:
  - url: "http://localhost:9094/api/v1/read"
```

## promQL

以下功能已从 promQL 中删除：

* `drop_common_labels`函数 - 应该使用`without`聚合修饰符
* `keep_common`聚合修饰符 - 应该是用`by`修饰符
* `count_scalar`函数 - `absent()`函数或在操作中正确传播标签可以更好地处理用例

有关更多详细信息，请参见 [issue \#3060](https://github.com/prometheus/prometheus/issues/3060)。

## 杂项 <a id="miscellaneous"></a>

### 非 root 用户运行 Prometheus <a id="prometheus-non-root-user"></a>

Prometheus Docker 镜像现在构建为[非 root 用户运行 Prometheus](https://github.com/prometheus/prometheus/pull/2859)。如果您希望 Prometheus UI/API 在低端口号\(如 80 端口\)上监听，则需要重写它。对于 Kubernetes，您可以使用如下 YAML：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-2
spec:
  securityContext:
    runAsUser: 0
...
```

更多详细信息，详见[为 Pod 或容器配置安全上下文](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

如果使用 Docker，将使用如下代码：

```bash
docker run -u root -p 80:80 prom/prometheus:v2.0.0-rc.2  --web.listen-address :80
```

### Prometheus 生命周期 <a id="prometheus-lifecycle"></a>

如果您使用 Prometheus `/-/reload` HTTP 端点在更改时[自动重载 Prometheus 配置](configuration/configuration.md)，出于安全原因在 Prometheus 2.0 默认禁用这些端点。要启用它们，设置`--web.enable-lifecycl`标志。

