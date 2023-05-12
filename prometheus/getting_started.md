# 快速开始

本指南是一种 "Hello World" 风格的教程，该教程演示了如何在简单的示例中安装、配置和使用 Prometheus。您将在本地下载并运行 Prometheus，对其进行配置并采集自身和示例应用程序数据，然后通过查询、规则和图形化的方式来使用采集到的时间序列数据。

## 下载并运行 Prometheus <a href="#downloading-and-running-prometheus" id="downloading-and-running-prometheus"></a>

下载适用于您的平台的[最新版本](https://prometheus.io/download)的 Prometheus，然后解压：

```bash
tar xvfz prometheus-*.tar.gz
cd prometheus-*
```

在启动 Prometheus 之前，我们先来配置它。

## 配置 Prometheus 监控自身 <a href="#configuring-prometheus-to-monitor-itself" id="configuring-prometheus-to-monitor-itself"></a>

Prometheus 通过 HTTP 访问目标端点的方式收集指标数据。由于 Prometheus 以相同的方式暴露其有关自身的数据，因此它可以采集并监控自身的运行状态。

虽然仅仅收集有关 Prometheus 自身数据在实践中并没有多大的用处，但它是一个好的开始示例。请将以下基本 Prometheus 配置保存为`prometheus.yml`文件：

```yaml
global:
  scrape_interval:     15s # 默认情况下，每 15s 采集一次目标数据

  # 与外部系统(如 federation, remote storage, Alertmanager)通信时，可以将这些标签应用到到和时间序列或告警上
  external_labels:
    monitor: 'codelab-monitor'

# 仅包含一个采集端点的采集配置：这里是 Prometheus 本身
scrape_configs:
  # 作业名称作为标签 `job=<job_name>` 添加到从此配置中采集的时间序列上
  - job_name: 'prometheus'

    # 覆盖全局默认的参数，并将采样时间间隔设置为 5s
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9090']
```

有关配置选项的完整说明，请参阅[配置文档](configuration/configuration.md)。

## 启动 Prometheus <a href="#starting-prometheus" id="starting-prometheus"></a>

要使用新创建的配置文件启动 Prometheus，请进入到包含有 Prometheus 二进制文件的目录，并执行如下命令：

```bash
# 启动 Prometheus
# 默认情况下，Prometheus 在 ./data 目录保存他的数据(参见 --storage.tsdb.path 参数)
./prometheus --config.file=prometheus.yml
```

Prometheus 应该正常启动了。您可以在浏览器上访问 [localhost:9090](http://localhost:9090)。Prometheus 需要几秒钟的时间通过其自身 HTTP 数据指标端点来采集数据。

您还可以通过访问其数据指标端点来验证 Prometheus 是否正在提供有关自身的指标：[localhost:9090/metrics](http://localhost:9090/metrics).

## 使用表达式浏览器 <a href="#using-the-expression-browser" id="using-the-expression-browser"></a>

我们来尝试查看 Prometheus 采集的有关自身的一些数据。要使用 Prometheus 的内置表达式浏览器，请导航至 [http://localhost:9090/graph](http://localhost:9090/graph) 并在 "Graph" 页面选择 "Console" 视图。

在 [localhost:9090/metrics](http://localhost:9090/metrics) 的数据采集中，Prometheus 导出了一个称为`prometheus_target_interval_length_seconds`(目标数据收集的实际时间)的数据指标。将其输入表达式控制台并点击 "Execute"：

```
prometheus_target_interval_length_seconds
```

这将会返回很多种不通的时间序列(及时间序列记录的最新值)，所有的时间序列的名称均为`prometheus_target_interval_length_seconds`，但是带有不同的标签。这些标签指定了不同的延迟百分比和和目标组间隔。

如果我们仅仅对第 99 个百分位的延迟感兴趣，则我们可以使用下面的语句来查询该信息：

```
prometheus_target_interval_length_seconds{quantile="0.99"}
```

想要统计返回时间序列的个数，您可以执行：

```
count(prometheus_target_interval_length_seconds)
```

有关表达式语言的更多信息，请见[表达式语言文档](querying/basics.md)

## 使用图形化接口 <a href="#using-the-graphing-interface" id="using-the-graphing-interface"></a>

请导航至 [http://localhost:9090/graph](http://localhost:9090/graph) 并使用 "Graph" 视图来使用图形表达式。

例如，输入以下表达式来绘制自身采集中P rometheus 每秒创建块的速率：

```bash
rate(prometheus_tsdb_head_chunks_created_total[1m])
```

在 graph 页面使用其它参数和设置进行实验。

## 启动示例目标采集数据 <a href="#starting-up-some-sample-targets" id="starting-up-some-sample-targets"></a>

让我们为 Prometheus 启动一些示例采集目标来变得更加有趣。

使用 Node Exporter 作为示例目标，有关使用它的更多信息参见[此指引](../guides/node-exporter.md)。

```bash
tar -xzvf node_exporter-*.*.tar.gz
cd node_exporter-*.*

./node_exporter --web.listen-address 127.0.0.1:8080
./node_exporter --web.listen-address 127.0.0.1:8081
./node_exporter --web.listen-address 127.0.0.1:8082
```

现在，您应该看到分别监听在 [http://localhost:8080/metrics](http://localhost:8080/metrics), [http://localhost:8081/metrics](http://localhost:8081/metrics), [http://localhost:8082/metrics](http://localhost:8082/metrics) 的目标示例。

## 配置 Prometheus 监控示例目标 <a href="#configuring-prometheus-to-monitor-the-sample-targets" id="configuring-prometheus-to-monitor-the-sample-targets"></a>

现在，我们将配置 Prometheus 来采集这些新的目标。我们将所有三个端点分组为 `node` 的作业。假设前两个端点是生产的目标，第三个端点代表金丝雀发布的实例。为了在 Prometheus 对此建模，我们可以将多个端点添加到单个作业中，并为每个目标组添加额外的标签。在此示例中，我们将`group="production"`标签添加到第一组目标中，将`group=canary`添加到第二组目标中。

为此，请将以下作业定义添加到`prometheus.yml`中的`scrape_configs`部分，然后重新启动 Prometheus 实例：

```yaml
scrape_configs:
  - job_name:       'node'

    # 覆盖全局默认的参数，并将采样时间间隔设置为 5s
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:8080', 'localhost:8081']
        labels:
          group: 'production'

      - targets: ['localhost:8082']
        labels:
          group: 'canary'
```

在表达式浏览器中验证 Prometheus 现在是否具有有关这些示例端点暴露的时间序列信息，如 `node_cpu_seconds_total` 数据指标。

## 配置规则将采集的数据汇总到新的时间序列中 <a href="#configure-rules-for-aggregating-scraped-data-into-new-time-series" id="configure-rules-for-aggregating-scraped-data-into-new-time-series"></a>

尽管在示例不是问题，但在临时计算时，汇总了数千个时间序列的查询可能会变慢。为了提高效率，Prometheus 允许通过配置的记录规则将表达式预记录到全新的持久时间序列中。假设我们想要记录在示例中测得的所有实例(保留`job`和`service`维度)在 5 分钟滑动窗口内的每秒 RPC 平均速率(`rpc_durations_seconds_count`)。我们可以这么写：

```
avg by (job, instance, mode) (rate(node_cpu_seconds_total[5m]))
```

尝试图形化此表达式。

要将由该表达式产生的时间序列记录到为名`job_instance_mode:node_cpu_seconds:avg_rate5m`的新数据指标中，请使用以下记录规则创建文件并保存为`prometheus.rules.yml`:

```yaml
groups:
- name: example	- name: cpu-node
  rules:	  rules:
  - record: job_service:rpc_durations_seconds_count:avg_rate5m	  - record: job_instance_mode:node_cpu_seconds:avg_rate5m
    expr: avg(rate(rpc_durations_seconds_count[5m])) by (job, service)	    expr: avg by (job, instance, mode) (rate(node_cpu_seconds_total[5m]))
```

要使 Prometheus 应用此新规则，需要在`prometheus.yml`中添加`rule_files`配置块。配置应如下所示：

```yaml
global:
  scrape_interval:     15s # 默认情况下，每 15s 采集一次目标数据
  evaluation_interval: 15s # 每 15s 进行一次规则评估

  # 与外部系统通信时，可以将这些标签应用到到和时间序列或告警上
  external_labels:
    monitor: 'codelab-monitor'

rule_files:
  - 'prometheus.rules.yml'

scrape_configs:
  - job_name: 'prometheus'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9090']

  - job_name:       'node'

    # 覆盖全局默认的参数，并将采样时间间隔设置为 5s
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:8080', 'localhost:8081']
        labels:
          group: 'production'

      - targets: ['localhost:8082']
        labels:
          group: 'canary'
```

通过新的配置重新启动 Prometheus，并通过表达式浏览器对其进行查询或画图，以验证数据指标名称为`job_instance_mode:node_cpu_seconds:avg_rate5m`的新时间序列已经可以使用。
