---
title: 使用基于文件的服务发现来发现数据采集目标
---

# 使用基于文件的服务发现来发现数据采集目标

Prometheus 提供了多种用于发现抓取目标的[服务发现选项](https://github.com/prometheus/prometheus/tree/master/discovery), 包括 [Kubernetes](../prometheus/configuration/configuration.md#kubernetes_sd_config), [Consul](../prometheus/configuration/configuration.md#consul_sd_config) 等。如果您使用系统当前不支持服务发现，则 Prometheus 的[基于文件的服务发现](../prometheus/configuration/configuration.md#file_sd_config)机制可能会最好地满足您的用例，您可以在 JSON 文件中列出采集目标\(以及有关这些目标的元数据\)。

在本指南中，我们将:

* 在本地安装并运行 Prometheus[Node Exporter](node-exporter.md)
* 创建 `targets.json` 文件，为 Node Exporter 指定主机和端口信息
* 安装并运行 Prometheus实例，该实例配置为使用 `targets.json` 文件发现 Node Exporter

## 安装并运行 Node Exporter <a id="installing-and-running-the-node-exporter"></a>

请参阅[使用 Node Exporter 监控 Linux 主机](node-exporter.md)的[安装](node-exporter.md#installing-and-running-the-node-exporter)部分。

节点导出器在 9100 端口上运行。为确保节点导出器公开指标:

```bash
curl http://localhost:9100/metrics
```

数据指标输出应该如下所示:

```text
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
go_gc_duration_seconds{quantile="0.25"} 0
go_gc_duration_seconds{quantile="0.5"} 0
...
```

## 安装配置并运行 Prometheus <a id="installing-configuring-and-running-prometheus"></a>

像 Node Exporter 一样，Prometheus 是一个静态二进制文件，可以通过 tar 包安装。[下载适合您平台的最新版](https://prometheus.io/download#prometheus) 并解压它:

```bash
wget https://github.com/prometheus/prometheus/releases/download/v*/prometheus-*.*-amd64.tar.gz
tar xvf prometheus-*.*-amd64.tar.gz
cd prometheus-*.*
```

解压目录包含 `prometheus.yml` 配置文件。使用如下内容替换该文件:

```yaml
scrape_configs:
- job_name: 'node'
  file_sd_configs:
  - files:
    - 'targets.json'
```

此配置指定存在一个名为 node 的作业\(用于Node Exporter\)，该作业从 `targets.json` 文件中检索 Node Exporter 实例的主机和端口信息。

This configuration specifies that there is a job called `node` \(for the Node Exporter\) that retrieves host and port information for Node Exporter instances from a `targets.json` file.

创建 `targets.json` 文件并将此内容添加到其中：

```javascript
[
  {
    "labels": {
      "job": "node"
    },
    "targets": [
      "localhost:9100"
    ]
  }
]
```

NOTE: 在本指南中，为了简洁起见，我们将手动使用 JSON 服务发现配置。 但是，一般而言，我们建议您改用某种 JSON 生成程序或工具。

此配置指定一个带有 `localhost:9100` 目标的名为 `node` 的作业

现在你可以启动 Prometheus:

```bash
./prometheus
```

如果 Prometheus 成功启动，您应该在日志中看到如下行:

```text
level=info ts=2018-08-13T20:39:24.905651509Z caller=main.go:500 msg="Server is ready to receive web requests."
```

## 公开发现服务的数据指标 <a id="exploring-the-discovered-services-metrics"></a>

随着 Prometheus 的启动和运行，您可以使用 Prometheus [表达式浏览器](../visualization/browser.md)检索 `node` 公开的指标。例如，如果您查询 \[[`up{job="node"}`](http://localhost:9090/graph?g0.range_input=1h&g0.expr=up%7Bjob%3D%22node%22%7D&g0.tab=1)  指标，则可以看到 Node Exporter 恰好被发现。

## 动态更改目标列表 <a id="changing-the-targets-list-dynamically"></a>

当使用Prometheus的基于文件的服务发现机制时，Prometheus实例将侦听对该文件的更改并自动更新抓取目标列表，而无需重新启动实例。 为了演示这一点，请在端口 9200 上启动另一个 Node Exporter 实例。首先导航到包含 Node Exporter 二进制文件的目录，然后在新的终端窗口中运行以下命令：

```bash
./node_exporter --web.listen-address=":9200"
```

现在，通过修改 `targets.json` 配置为新的 Node Exporter 添加一个条目:

```javascript
[
  {
    "targets": [
      "localhost:9100"
    ],
    "labels": {
      "job": "node"
    }
  },
  {
    "targets": [
      "localhost:9200"
    ],
    "labels": {
      "job": "node"
    }
  }
]
```

保存更改后，Prometheus 将自动收到新目标列表的通知。[`up{job="node"}`](http://localhost:9090/graph?g0.range_input=1h&g0.expr=up%7Bjob%3D%22node%22%7D&g0.tab=1) 数据指标将显示`instance` 标签分别为 `localhost:9100` 和 `localhost:9200` 的两个实例，

## 小结 <a id="summary"></a>

在本指南中，您安装并运行了 Prometheus Node Exporter，并配置了 Prometheus 使用基于文件的服务发现从 Node Exporter 中发现和采集指标。

