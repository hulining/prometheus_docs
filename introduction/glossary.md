---
title: 相关术语
---

# 相关术语

## Alert

告警是 Prometheus 中积极触发的警报规则的结果。警报从 Prometheus 发送到 Alertmanager

## Alertmanager

[Alertmanager](overview.md) 接收告警，将告警分组组，重复数据删除，应用静默，限制，然后将通知发送到电子邮件，Pagerduty，Slack 等

## Bridge

Bridge 是从客户端库中获取样本并将其暴露给非 Prometheus 监控系统的组件。例如，Python，Go 和 Java 客户端可以将指标数据导出到 Graphite。

## Client library

客户端库是某种语言\(如Go，Java，Python，Ruby\)的库，可以轻松地直接检测代码，编写自定义收集器以便于从其他系统提取指标并将指标暴露给 Prometheus。

## Collector

Collector 是代表一组指标的 exporter 的一部分。 如果它是直接检测的一部分，则可能是单个数据指标，如果是从另一个系统提取数据指标，则可能是多个数据指标。

## Direct instrumentation

Direct instrumentation 是使用客户端库内联作为程序源代码的一部分内联添加的检测。

## Endpoint

可以采集数据指标的源地址，通常对应于独立的程序。

## Exporter

Exporter 是与您要从中获取数据指标的应用程序一起运行的二进制文件。Exporter 程序通常通过将非 Prometheus 格式数据指标转换为 Prometheus 支持的格式来暴露 Prometheus 指标。

## Instance

Instance 是作业中目标标签的唯一标识。

## Job

具有相同目的的目标的集合\(例如，监视一组为可伸缩性或可靠性而复制的相似过程\)被称为作业。

## Notification

代表一组一个或多个告警，并由 Alertmanager 发送到电子邮件，Pagerduty，Slack等。

## Promdash

Promdash 是 Prometheus 的本地仪表盘构建器。它已被弃用，并被 Grafana 取代。

## Prometheus

通常是指 Prometheus 系统的核心二进制文件。它也可能是整个 Prometheus 监控系统。

## PromQL

[PromQL](../prometheus/querying/basics.md) 是 Prometheus 的查询语言。它允许进行多种操作，包括聚合，切片和切块，预测和连接

## Pushgateway

[Pushgateway](../instrumenting/pushing.md) 保留了批处理作业中最新推送的指标。这使 Prometheus 可以在它们终止后仍可以采集数据指标。

## Remote Read

远程读取是 Prometheus 的一项功能，该功能允许从其它系统\(如长期存储\)读取时间序列。

## Remote Read Adapter

并非所有系统都直接支持远程读取。远程读取适配器位于 Prometheus 和另一个系统之间，可以在它们之间转换时间序列请求和响应。

## Remote Read Endpoint

Prometheus 进行远程读取时要与之通信的端点。

## Remote Write

远程写是Prometheus的一项功能，它允许将摄取的样本即时发送到其他系统，例如长期存储。

## Remote Write Adapter

并非所有系统都直接支持远程写入。远程写适配器位于 Prometheus 和另一个系统之间，将远程写操作中的样本转换为另一个系统可以理解的格式。

## Remote Write Endpoint

Prometheus 执行远程写时要与之通信的端点。

## Sample

时间序列中某个时间点的单个值。

在Prometheus中，每个数据样本都包含一个float64值和一个毫秒精度的时间戳。

## Silence

Alertmanager 中的静默功能可防止带有与静默匹配的标签的警报被包含在通知中。

## Target

对要采集数据指标对象的定义。例如，要应用的标签，连接所需的任何身份验证或定义采集方式等信息。

