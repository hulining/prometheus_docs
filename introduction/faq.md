---
title: 常见问题
---

# 常见问题

## 一般问题 <a id="general"></a>

### Prometheus 是什么？ <a id="what-is-prometheus"></a>

Prometheus 是具有活跃生态系统的开源系统监视和警报工具包。请参阅[概述](overview.md)。

### Prometheus 与其他监控系统相比如何？ <a id="how-does-prometheus-compare-against-other-monitoring-systems"></a>

请参阅[比较](comparison.md)页面。

### Prometheus 有哪些依赖？ <a id="what-dependencies-does-prometheus-have"></a>

Prometheus 主服务独立运行，没有外部依赖。

### Prometheus 可以配置为高可用吗？ <a id="can-prometheus-be-made-highly-available"></a>

可以，在两台或多台独立的机器上运行相同的 Prometheus 服务。相同的告警将由 [Alertmanager](https://github.com/prometheus/alertmanager) 进行重复数据删除。

关于 [Alertmanager 的高可用](https://github.com/prometheus/alertmanager#high-availability)，您可以在 [Mesh 集群](https://github.com/weaveworks/mesh)中运行多个实例，并配置 Prometheus 服务器向每个实例发送通知。

### Prometheus 没有扩展和收缩 <a id="i-was-told-prometheus-doesnt-scale"></a>

实际上，存在多种缩放和联合 Prometheus 的方法。阅读 Robust Perception 的 [Scaling and Federating Prometheus](https://www.robustperception.io/scaling-and-federating-prometheus/) 博客开始使用。

### Prometheus 使用什么语言编写而成？ <a id="what-language-is-prometheus-written-in"></a>

大多数 Prometheus 组件使用 Go 编写的。有些使用 Java，Python 和 Ruby 编写。

### Prometheus 功能，存储格式和 API 的稳定性如何？ <a id="how-stable-are-prometheus-features-storage-formats-and-apis"></a>

Prometheus GitHub 组织中所有已达到版本 1.0.0 的仓库都大致遵循[语义版本控制](http://semver.org/)。 重大更改以主要版本的增量表示。实验性组件可能会出现例外，声明中会明确标明例外情况。

通常，即使尚未达到 1.0.0 版的仓库也相当稳定。我们的目标是为每个仓库制定适当的发布流程并最终发布 1.0.0。在任何情况下，都会在发行说明中指出重大更改\(`由[CHANGE]标记`\)，或者对于尚未正式发行的组件进行明确传达。

### 为什么使用数据拉取方式而不是数据推送方式？ <a id="why-do-you-pull-rather-than-push"></a>

通过 HTTP 拉取数据的方式有很多优点：

* 开发更改时，可以在笔记本电脑上运行监控。
* 您可以更轻松地判断目标是否已关闭。
* 您可以手动转到目标并使用 Web 浏览器检查其运行状况。

总体而言，我们认为拉取数据比推送数据略好，但在考虑使用监控系统时，不应将其视为重点。

对于必须推送的情况，我们提供了 [Pushgateway](../instrumenting/pushing.md)。

### Prometheus 如何处理日志？ <a id="how-to-feed-logs-into-prometheus"></a>

简短的回答：不要使用 Prometheus 处理日志！请改用类似于 [ELK 栈](https://www.elastic.co/products)工具

更长的答案：Prometheus 是一个收集和处理指标的系统，而不是事件记录系统。Raintank 博客文章 [Logs and Metrics and Graphs，Oh My！](https://blog.raintank.io/logs-and-metrics-and-graphs-oh-my/)提供了有关日志和指标之间差异的更多详细信息。

如果您想从应用程序日志中提取 Prometheus 指标，Google 的 [mtail](https://github.com/google/mtail) 可能会有所帮助。

### Prometheus 是谁写的？ <a id="who-wrote-prometheus"></a>

Prometheus 最初由 [Matt T. Proud](http://www.matttproud.com/) 和 [Julius Volz](http://juliusv.com/) 私人创立。它的大部分初始开发是由 [SoundCloud](https://soundcloud.com/) 赞助的。

现在，它已由众多公司和个人维护和扩展。

### Prometheus 基于什么协议？ <a id="what-license-is-prometheus-released-under"></a>

Prometheus 是根据 [Apache 2.0](https://github.com/prometheus/prometheus/blob/master/LICENSE) 许可发布的。

### Prometheus 的复数是什么？ <a id="what-is-the-plural-of-prometheus"></a>

经过[广泛的研究](https://youtu.be/B_CDeYrqxjQ)，已经确定 "Prometheus" 的正确复数是 "Prometheis"。

### 我可以重新加载 Prometheus 的配置吗？ <a id="can-i-reload-prometheuss-configuration"></a>

可以，将`SIGHUP`信号发送到 Prometheus 进程或将 HTTP POST请求发送到`/-/reload` 端点将重新加载并应用配置文件。各种组件将尝试优雅地处理失败的更改。

### 可以发送告警吗？ <a id="can-i-send-alerts"></a>

可以，使用 [Alertmanager](https://github.com/prometheus/alertmanager)。

目前，支持以下外部系统发送告警信息：

* 邮件
* 通用 webhook
* [HipChat](https://www.hipchat.com/)
* [OpsGenie](https://www.opsgenie.com/)
* [PagerDuty](https://www.pagerduty.com/)
* [Pushover](https://pushover.net/)
* [Slack](https://slack.com/)

### 可以创建仪表板吗？ <a id="can-i-create-dashboards"></a>

可以，我们建议您使用 [Grafana](../visualization/grafana.md) 用于生产。也有内置的[控制台模板](../visualization/consoles.md)。

### 可以更改时区吗？为什么所有内容都采用 UTC 时间？ <a id="can-i-change-the-timezone-why-is-everything-in-utc"></a>

为避免任何形式的时区混乱，特别是在涉及所谓的夏时制时，我们决定在 Prometheus 的所有组件中内部专门使用 Unix time 和 UTC 进行显示。可以将精心完成的时区选择引入 UI。 欢迎贡献。 有关此工作的当前状态，请参阅 [issue \#500](https://github.com/prometheus/prometheus/issues/500)。

## 工具 <a id="instrumentation"></a>

### 有哪些语言的工具库? <a id="which-languages-have-instrumentation-libraries"></a>

有许多客户端库可使用 Prometheus 指标来检测您的服务。有关详细信息，请参见[客户端库](../instrumenting/clientlibs.md)文档。

如果您有兴趣贡献新语言的客户库，请参见[exposition formats](../instrumenting/exposition_formats.md)

### 可以监控机器吗？ <a id="can-i-monitor-machines"></a>

可以，[Node Exporter](https://github.com/prometheus/node_exporter) 在 Linux 和其他 Unix 系统上暴露了大量的计算机级别指标，例如CPU使用率，内存，磁盘使用率，文件系统完整性和网络带宽。

### 可以监视网络设备吗？ <a id="can-i-monitor-network-devices"></a>

可以，[SNMP Exporter](https://github.com/prometheus/snmp_exporter) 允许监控支持 SNMP 的设备。

### 可以监控批处理作业吗？ <a id="can-i-monitor-batch-jobs"></a>

可以，使用 [Pushgateway](../instrumenting/pushing.md)。另请参阅监视批处理作业的[最佳实践](../practices/instrumentation.md#batch-jobs)。

### Prometheus 可以直接监控哪些应用程序？ <a id="what-applications-can-prometheus-monitor-out-of-the-box"></a>

请参阅[导出器及集成的列表](../instrumenting/exporters.md)。

### 可以通过 JMX 监控 JVM 应用程序吗？ <a id="can-i-monitor-jvm-applications-via-jmx"></a>

可以，对于不能直接使用 Java 客户端进行检测的应用程序，可以将 [JMX Exporter](https://github.com/prometheus/jmx_exporter) 单独使用或作为 Java 代理使用。

### 什么影响工具的性能？ <a id="what-is-the-performance-impact-of-instrumentation"></a>

客户端库和语言之间的性能可能会有所不同。对于 Java，[基准测试](https://github.com/prometheus/client_java/blob/master/benchmark/README.md)表明，取决于争用情况，使用Java客户端增加计数器/表将花费 12-17ns。除了对延迟最关键的代码之外，所有其他代码都可以忽略不计。

## 故障排查 <a id="troubleshooting"></a>

### Prometheus 1.x 服务需要花费很长时间才能启动，日志中显示有大量崩溃恢复的信息 <a id="my-prometheus-1-x-server-takes-a-long-time-to-start-up-and-spams-the-log-with-copious-information-about-crash-recovery"></a>

不完全关闭造成的。在接收到`SIGTERM`信号之后，Prometheus 必须彻底关闭，这对于频繁使用的服务器可能需要一段时间。如果服务器崩溃或被强制杀死\(例如，由于 OOM 被内核杀死或在等待 Prometheus 关闭时，您的运行级别系统不耐烦\)，则必须执行崩溃恢复，在正常情况下，恢复时间应少于一分钟，但在某些情况下要花很长时间。有关详细信息，请参见[崩溃恢复](https://prometheus.io/docs/prometheus/1.8/storage/#crash-recovery)。

### Prometheus 1.x 服务内存不足 <a id="my-prometheus-1-x-server-runs-out-of-memory"></a>

请参阅有关配置 [Prometheus 的内存使用情况](https://prometheus.io/docs/prometheus/1.8/storage/#memory-usage)的部分以获取可用内存量。

### Prometheus 1.x 服务报告处于 "紧急模式" 或 "存储需要节流" <a id="my-prometheus-1-x-server-reports-to-be-in-rushed-mode-or-that-storage-needs-throttling"></a>

您的存储设备负担重。阅读有关[配置本地存储的部分](https://prometheus.io/docs/prometheus/1.8/storage/)，以了解如何调整设置以获得更好的性能。

## 实践 <a id="implementation"></a>

### 为什么所有样本值都是 64 位浮点数？我想要整数 <a id="why-are-all-sample-values-64-bit-floats-i-want-integers"></a>

我们将自己限制在 64 位浮点数以简化设计。[IEEE 754 双精度二进制浮点格式](https://en.wikipedia.org/wiki/Double-precision_floating-point_format)支持最大精度为`2^53`的整数精度。仅当需要大于$`2^53`但小于`2^63`的整数精度时，支持本机 64 位整数\(仅\)会有所帮助。原则上，支持不同的样本值类型\(包括某种大整数，甚至支持64位以上\)都可以实现，但目前不是优先事项。计数器即使每秒增加一百万次，也只会在超过 285 年后才会出现精度问题。

### 为什么 Prometheus 服务组件不支持 TLS 或身份验证？我可以添加那些吗？ <a id="why-dont-the-prometheus-server-components-support-tls-or-authentication-can-i-add-those"></a>

注意：Prometheus 团队在 2018 年 8 月 11 日的开发峰会上已改变了立场，该[项目的路线图](roadmap.md)现已支持对 TLS 和服务端点中的身份验证的支持。更改代码后，将更新此文档。

尽管 TLS 和身份验证是经常需要的功能，但我们故意没有在 Prometheus 的任何服务端组件中实现它们。两者都有太多不同的选项和参数\(仅TLS就有 10 多个选项\)，我们决定专注于构建最佳监视系统，而不是在每个服务器组件中都支持完全通用的 TLS 和身份验证解决方案。

如果您需要 TLS 或身份验证，我们建议在 Prometheus 前面放置一个反向代理。例如，参见[使用 Nginx 向 Prometheus 添加基本身份验证](https://www.robustperception.io/adding-basic-auth-to-prometheus-with-nginx/)。

这仅适用于入站连接。 Prometheus 支持[从启用 TLS 和 auth 的目标采集数据](../prometheus/configuration/configuration.md#tls_config)，并且其他创建出站连接的 Prometheus 组件也具有类似的支持。

