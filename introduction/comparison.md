---
title: 与替代品比较
---

# 与替代品比较

## Prometheus vs. Graphite

### 适用范围

[Graphite](https://graphite.readthedocs.org/en/latest/) 专注于成为具有查询语言和图形功能的消极时间序列数据库。其他任何顾虑都通过外部组件解决

Prometheus 是一个完整的监控和趋势分析系统，其中包括基于时间序列数据的内置主动采集，存储，查询，制图和告警。它掌握系统的信息\(应该存在哪些端点，什么时间序列模式意味着麻烦等\)，并积极尝试查找错误。

### 数据模型

Graphite 像Prometheus一样存储命名时间序列的数值样本。但 Prometheus 的元数据模型更加丰富：尽管 Graphite 指标名称由点分隔的多部分组成，这些部分隐式地对维进行编码，但是 Prometheus 明确将维度编码为键值对\(称为标签\)，并附加到指标名称。这允许通过查询语言通过这些标签轻松进行过滤，分组和匹配。

此外，尤其是当将 Graphite 与 [StatsD](https://github.com/etsy/statsd/) 结合使用时，通常只在所有受监视实例上存储聚合数据，而不是将实例保留为一个维度并能够深入分析单个有问题的实例。

例如，使用 Graphite/StatsD 来对请求到 API 服务器 `/tracks` 端点的 `POST` 方法且响应状态码为 `500` 的 HTTP 请求数进行编码：

```text
stats.api-server.tracks.post.500 -> 93
```

在 Prometheus 中，可以像这样对相同的数据进行编码\(假设三个 api-server 实例\)：

```text
api_server_http_requests_total{method="POST",handler="/tracks",status="500",instance="<sample1>"} -> 34
api_server_http_requests_total{method="POST",handler="/tracks",status="500",instance="<sample2>"} -> 28
api_server_http_requests_total{method="POST",handler="/tracks",status="500",instance="<sample3>"} -> 31
```

### 存储

Graphite 以 [Whisper](https://graphite.readthedocs.org/en/latest/whisper.html) 格式将时间序列数据存储在本地磁盘上，这是一种 RRD 样式的数据库，它希望样本以固定的时间间隔到达。 每个时间序列都存储在一个单独的文件中，新样本在一定时间后会覆盖旧样本。

Prometheus 也会在每个时间序列上创建一个本地文件，但允许在采集数据或规则评估时以任意间隔存储样本。由于仅附加了新样本，因此旧数据可以任意保留。Prometheus 也适用于许多短暂的，经常变化的时间序列集。

### 小结

Prometheus 除了更易于运行和集成到您的环境之外，还提供了更丰富的数据模型和查询语言。如果您想要一个可以长期保存历史数据的群集解决方案，那么 Graphite 可能是一个更好的选择。

## Prometheus vs. InfluxDB

[InfluxDB](https://influxdata.com/) 是一个开放源代码的时间序列数据库，具有用于扩展和集群化的商业选项。 Prometheus 开发开始将近一年后，InfluxDB 项目才发布，因此我们当时无法将其视为替代方案。尽管如此，Prometheus 和 InfluxDB 之间仍然存在显着差异，并且两种系统面向的用例稍有不同。

### 适用范围

为了进行公平的比较，我们需要将 [Kapacitor](https://github.com/influxdata/kapacitor) 与 InfluxDB 一起考虑，因为它们结合起来可以解决与 Prometheus 和 Alertmanager 相同的问题。

对于 InfluxDB 本身，与 Graphite 适用范围的差异同样适用于此。此外，InfluxDB 还提供连续查询，这些查询等同于 Prometheus 记录规则。

Kapacitor 作用范围是 Prometheus 记录规则，警报规则和 Alertmanager 的通知功能的组合。Prometheus 提供了[更强大的查询语言来进行图形显示和警报](https://www.robustperception.io/translating-between-monitoring-languages/)。Prometheus Alertmanager 还提供了分组，重复数据删除和静音功能。

### 数据模型/存储

与 Prometheus 一样，InfluxDB 数据模型也将键值对作为标签，称为 tags。此外，InfluxDB 还有第二级标签，称为 fields，使用范围受到更多限制。InfluxDB 支持高达十亿分之一秒分辨率的时间戳，以及 float64，int64，bool 和字符串数据类型。相比之下，Prometheus 支持 float64 数据类型，但对字符串和毫秒分辨率的时间戳支持有限。

InfluxDB 使用[日志结构合并树的变体作为带有按时间分片的预写日志的存储](https://docs.influxdata.com/influxdb/v1.7/concepts/storage_engine/)。与 Prometheus 每个时间序列的仅附加文件相比，此方法更适合事件记录。

博客[Logs and Metrics and Graphs, Oh My! ](https://blog.raintank.io/logs-and-metrics-and-graphs-oh-my/)描述了事件日志和度量记录之间的差别。

### 架构

Prometheus 服务器彼此独立运行，并且依靠其本地存储来实现其核心功能：数据采集，规则处理和告警。InfluxDB 的开源版本与此类似。

商业 InfluxDB 产品是一个分布式存储集群，其中存储和查询由多个节点一次处理。

这意味着商业 InfluxDB 将更易水平扩展，但这也意味着您必须从一开始就管理分布式系统的复杂性。Prometheus 的运行更简单，但在某些时候，您将需要按照产品，服务，数据中心或类似的可伸缩行边界明确地分片服务器。独立服务\(可以并行冗余运行\)也可以为您提供更好的可靠性和故障隔离。

Kapacitor 的开源版本没有用于规则，警报或通知的内置分布式/冗余选项。Kapacitor 的开源发行版可以通过用户手动分片来扩展，类似于 Prometheus 本身。Influx提供了 [Enterprise Kapacitor](https://docs.influxdata.com/enterprise_kapacitor)，它支持高可用警报系统。

相比之下，Prometheus 和 Alertmanager 通过运行 Prometheus 的冗余副本并使用 Alertmanager 的[高可用](https://github.com/prometheus/alertmanager#high-availability)模式提供了完全开源的冗余选项。

### 小结

两个系统之间有很多相似之处。两者都有标签\(在 InfluxDB 称为 tags\)，可以有效地支持多维度指标数据。两者都使用基本相同的数据压缩算法。两者都有广泛的集成，包括彼此之间的集成。两者都有可以让您进一步扩展的钩，例如使用统计工具分析数据或执行自动操作。

InfluxDB 的优点：

* 进行事件记录。
* 商业版本为 InfluxDB 提供集群服务，这对长期数据存储也更好。
* 在副本之间保持最终一致的数据视图。

Prometheus 的优点

* 主要用于做指标数据
* 更强大的查询语言，告警和通知功能
* 图形和告警的可用性和正常运行时间更高

InfluxDB 由一家商业公司按照开放核心模型进行维护，并提供高级功能，如闭源的群集，托管和支持。Prometheus 是一个完全开源的独立项目，由许多公司和个人维护，其中一些也提供商业服务和支持。

## Prometheus vs. OpenTSDB

[OpenTSDB](http://opentsdb.net/) 是基于 [Hadoop](https://hadoop.apache.org/) 和 [HBase](https://hbase.apache.org/) 的分布式时间序列数据库。

### 适用范围

与 Graphite 适用范围的差异同样适用于此。

### 数据模型

OpenTSDB 的数据模型几乎与 Prometheus 的数据模型相同：时间序列由一组任意键值对标识\(OpenTSDB 中为 tags\)。指标的所有数据[存储在一起](http://opentsdb.net/docs/build/html/user_guide/writing/index.html#time-series-cardinality)，从而限制了指标的基数。尽管有一些细微的差别：Prometheus 允许标签值中包含任意字符，而OpenTSDB 的限制更严格。OpenTSDB 还缺少完整的查询语言，仅允许通过其 API 进行简单的汇总和数学运算。

### 存储

[OpenTSDB](http://opentsdb.net/) 的存储在 [Hadoop](https://hadoop.apache.org/) 和 [HBase](https://hbase.apache.org/) 之上实现。这意味着可以轻松水平扩展 OpenTSDB，但您必须从一开始就接受 Hadoop/HBase 集群总体的复杂性。

Prometheus 最初运行起来会更简单，但是一旦超出单个节点的容量，就需要进行明确的分片。

### 小结

Prometheus 提供了更丰富的查询语言，可以处理更高的基数指标，并且构成了完整监视系统的一部分。如果您已经在运行 Hadoop 并重视长期存储的优势，那么 OpenTSDB 是一个不错的选择。

## Prometheus vs. Nagios

Nagios 是一个始于 1990 年代的 NetSaint 监控系统。

### 适用范围

Nagios 主要是基于脚本的退出代码进行警报。这些称为 "checks"。单个警报会静音，但是不会进行分组，路由或重复数据删除。

Nagios 有各种各样的插件。例如，perfData 插件允许使用管道传输将几千字节的数据返回到[时间序列数据库，如 Graphite](https://github.com/shawn-sterling/graphios)或使用NRPE [在远程计算机上运行检查](https://exchange.nagios.org/directory/Addons/Monitoring-Agents/NRPE--2D-Nagios-Remote-Plugin-Executor/details)。

### 数据模型

Nagios 是基于主机的。每个主机可以具有一个或多个服务，并且每个服务可以执行一项检查。Nagios 没有标签或查询语言的概念。

### 存储

除了当前的检查状态外，Nagios 本身没有存储空间。有一些插件可以存储诸如[可视化](https://docs.pnp4nagios.org/)的数据。

### 架构

Nagios 服务是独立的，所有的检查配置均通过文件进行。

### 小结

Nagios 适用于黑匣子探测已足够的小型或静态系统的基本监控。

如果您想进行白盒监控，或者具有动态或基于云的环境，那么 Prometheus 是一个不错的选择

## Prometheus vs. Sensu

[Sensu](https://sensu.io/) 是可组合的监控管道，可以重用现有的 Nagios 检查

### 适用范围

与 Nagios 适用范围的差异同样适用于此。

Sensu 包含一个[客户端套接字](https://docs.sensu.io/sensu-core/latest/reference/clients/#what-is-the-sensu-client-socket)，允许将临时检查结果推送到 Sensu 中。

### 数据模型

Sensu 与 Nagios 具有相同粗略的数据模型。

### 存储

Sensu 使用 Redis 持久化监视数据，包括 Sensu 客户端注册，检查结果，检查执行历史记录和当前事件数据。

### 架构

Sensu 具有[许多组件](https://docs.sensu.io/sensu-core/latest/overview/architecture/)。它使用 RabbitMQ 作为传输工具，使用 Redis 记录当前状态，并使用单独的服务器进行处理和 API 访问。

Sensu部署的所有组件\(RabbitMQ，Redis 和 Sensu Server/API\)都可以集群化，以实现高可用性和冗余配置。

### 小结

如果您现有的 Nagios 设置希望按原样缩放，或者想利用 Sensu 的自动注册功能，那么 Sensu 是一个不错的选择。

如果您想进行白盒监控，或者具有非常动态或基于云的环境，那么Prometheus是一个不错的选择。

