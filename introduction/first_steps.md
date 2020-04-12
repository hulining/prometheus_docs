---
title: 初识 Prometheus
---

# 初识 Prometheus

欢迎来到 Prometheus！Prometheus 是一个通过从监控目标上采集 HTTP 数据收集指标数据的监控平台。本指南将向您展示如果安装、配置 Prometheus，并使用 Prometheus 监控第一个资源对象。您将下载、安装并运行 Prometheus。您也可以下载并安装一个可在主机和服务上显示时间序列数据的 exporter 工具。我们的第一个 exporter 将是 Prometheus 本身，它提供有关内存使用，垃圾回收等各种主机级别的指标。

## 下载 Prometheus <a id="downloading-prometheus"></a>

下载适用于您的平台的[最新版本的 Prometheus](https://prometheus.io/download)，然后解压：

```bash
tar xvfz prometheus-*.tar.gz
cd prometheus-*
```

Prometheus server 是一个名为`prometheus`的二进制文件\(在 Windows 为`prometheus.exe`\)。我们可以运行二进制文件，并通过传入`--help`参数来查看其帮助

```bash
./prometheus --help
usage: prometheus [<flags>]

The Prometheus monitoring server
. . .
```

在启动 Prometheus 之前，我们先来配置它。

## 配置 Prometheus <a id="configuring-prometheus"></a>

Prometheus 的配置为 [YAML](http://www.yaml.org/start.html)。Prometheus 的下载文件中包含一个名为`prometheus.yml`的示例配置，我们可以根据它来开始配置。

我们删除了示例配置中的大多数注释，使其更加简洁\(注释是以`#`开头的行 \)

```yaml
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
```

示例配置文件中包含三个配置块：`global`，`rule_files`和`scrape_configs`。

`global`配置块控制 Prometheus server 的全局配置。目前有两个参数。`scrape_interval`参数控制 Prometheus 多久采集一次 targets。您可以为单个 targets 重新配置此参数。在示例中，全局配置为 15 秒采集一次。`evaluation_interval` 参数控制 Prometheus 多久评估一次 rules。Prometheus 使用 rules 来创建新的时间序列并生成报警。

`rule_files`配置块指定我们希望 Prometheus server 加载规则的文件位置。目前，我们没有任何规则。

`scrape_configs`配置块控制 Prometheus 监控哪些资源。由于 Prometheus 将有关自身的数据暴露为 HTTP 端点，因此它可以采集并监控其自身的运行状况。在默认配置中，只有一个名为`prometheus`的作业，它会采集 Prometheus 服务暴露的时间序列数据。该作业包含一个单独的静态配置的 target，为`localhost`，`9090`端口。Prometheus 希望数据 指标可以通过 target 的`/metrics`路径可以访问。所以这个默认作业通过`http://localhost:9090/metrics`进行指标数据采集。

返回的时间序列数据将详细说明 Prometheus server 的状态和性能。

有关配置选项的完整说明，请参阅[配置文档](../prometheus/configuration/configuration.md)

## 启动 Prometheus <a id="starting-prometheus"></a>

要使用我们新创建的配置文件启动 Prometheus，请切换到包含 Prometheus 二进制文件的目录并运行：

```bash
./prometheus --config.file=prometheus.yml
```

Prometheus 应该能够启动成功。您应该能够在`http://localhost:9090`上看到有关其自身状态的页面。它需要大约 30 秒的时间从自己的 HTTP 指标数据端点收集有关自身的数据。

你也可以通过访问 Prometheus 自身的指标数据端点`http://localhost:9090/metrics`来验证 Prometheus 是否正在提供有关其自身数据的指标数据。

## 使用表达式浏览器 <a id="using-the-expression-browser"></a>

让我们尝试查看 Prometheus 收集的有关自身的一些数据。要使用 Prometheus 的内置表达式浏览器，请导航至`http://localhost:9090/graph`，然后在 "Graph" 选项卡中选择 "Console" 视图。

正如您可以从`http://localhost:9090/metrics`获取的数据一样，Prometheus 暴露一个称为`promhttp_metric_handler_requests_total`\(Prometheus 已处理`/metrics`的请求总数\)的指标数据。将其输入到表达式控制台中：

```text
promhttp_metric_handler_requests_total
```

它应该返回多个不同的时间序列\(以及每个时间序列的最新值\)，所有时间序列的 metric 名称均为 `promhttp_metric_handler_requests_total`，但具有不同的标签。这些标签指定了不同的请求状态。

如果我们只对 HTTP 状态码为`200`的请求感兴趣，则可以使用此查询来检索该信息：

```text
promhttp_metric_handler_requests_total{code="200"}
```

要计算返回的时间序列数，您可以查询

```text
count(promhttp_metric_handler_requests_total)
```

有关表达语言的更多信息，请参见[表达语言文档](../prometheus/querying/basics.md)

## 使用绘图界面 <a id="using-the-graphing-interface"></a>

要使用绘制图形表达式，请导航至`http://localhost:9090/graph`并使用 "Graph" 选项卡。

例如，输入以下表达式以图形化显示从自身采集的返回状态代码为 200 的 HTTP 请求的速率：

```text
rate(promhttp_metric_handler_requests_total{code="200"}[1m])
```

您可以尝试其它参数和设置使用图形页面

## 监控其它 target <a id="monitoring-other-targets"></a>

仅从 Prometheus 收集指标并不能很好地说明 Prometheus 的功能。为了更好地了解 Prometheus 可以做什么，我们建议您浏览有关其它 exporter 的文档。 使用 [node\_exporter](../guides/node-exporter.md) 监控 Linux 或 macOS 是一个不错的选择。

## 总结 <a id="summary"></a>

在本指南中，您安装了 Prometheus，配置了 Prometheus 实例以监视资源，并了解了在 Prometheus 表达式浏览器中使用时间序列数据的一些基础知识。 要继续学习 Prometheus，请查看[概述](overview.md)以获取有关接下来要探索的内容的一些想法。

