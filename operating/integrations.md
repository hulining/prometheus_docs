---
title: 相关集成
---

# 集成

除了[客户端库](../instrumenting/clientlibs.md)、[exporter 和相关集成](../instrumenting/exporters.md)之外，Prometheus 还有许多其他通用集成。此页面列出了与这些产品的一些集成。

由于某些集成组件功能重复或仍在开发中，因此未在此处列出所有集成。[exporter 默认端口](https://github.com/prometheus/prometheus/wiki/Default-port-allocations) Wiki 页面也恰好包括一些符合这些类别的不是 exporter 的集成。

## 文件服务发现 <a id="file-service-discovery"></a>

对于 Prometheus 本身不支持的服务发现机制，基于文件的服务发现提供了集成接口。

* [Docker Swarm](https://github.com/ContainerSolutions/prometheus-swarm-discovery)
* [Kuma](https://github.com/Kong/kuma/tree/master/app/kuma-prometheus-sd)
* [Lightsail](https://github.com/n888/prometheus-lightsail-sd)
* [Packet](https://github.com/packethost/prometheus-packet-sd)
* [Scaleway](https://github.com/scaleway/prometheus-scw-sd)

## 远程端点和存储 <a id="remote-endpoints-and-storage"></a>

Prometheus 的[远程写入](https://prometheus.io/docs/operating/configuration/#remote_write)和[远程读取](https://prometheus.io/docs/operating/configuration/#remote_read)功能允许透明地发送和接收数据样本。这主要用于长期存储。建议您仔细评估此空间中的任何解决方案，以确认它可以承载您的数据存储。

* [AppOptics](https://github.com/solarwinds/prometheus2appoptics): 写
* [Azure Data Explorer](https://github.com/cosh/PrometheusToAdx): 读写
* [Azure Event Hubs](https://github.com/bryanklewis/prometheus-eventhubs-adapter): 写
* [Chronix](https://github.com/ChronixDB/chronix.ingester): 写
* [Cortex](https://github.com/cortexproject/cortex): 读写
* [CrateDB](https://github.com/crate/crate_adapter): 读写
* [Elasticsearch](https://github.com/infonova/prometheusbeat): 写
* [Gnocchi](https://gnocchi.xyz/prometheus.html): 写
* [Google Cloud Spanner](https://github.com/google/truestreet): 读写
* [Graphite](https://github.com/prometheus/prometheus/tree/master/documentation/examples/remote_storage/remote_storage_adapter): 写
* [InfluxDB](https://docs.influxdata.com/influxdb/latest/supported_protocols/prometheus): 读写
* [IRONdb](https://github.com/circonus-labs/irondb-prometheus-adapter): 读写
* [Kafka](https://github.com/Telefonica/prometheus-kafka-adapter): 写
* [M3DB](https://m3db.github.io/m3/integrations/prometheus): 读写
* [OpenTSDB](https://github.com/prometheus/prometheus/tree/master/documentation/examples/remote_storage/remote_storage_adapter): 写
* [PostgreSQL/TimescaleDB](https://github.com/timescale/prometheus-postgresql-adapter): 读写
* [QuasarDB](https://doc.quasardb.net/master/user-guide/integration/prometheus.html): 读写
* [SignalFx](https://github.com/signalfx/metricproxy#prometheus): 写
* [Splunk](https://github.com/kebe7jun/ropee): 读写
* [TiKV](https://github.com/bragfoo/TiPrometheus): 读写
* [Thanos](https://github.com/thanos-io/thanos): 写
* [VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics): 写
* [Wavefront](https://github.com/wavefrontHQ/prometheus-storage-adapter): 写

## Alertmanager Webhook Receiver

对于 Alertmanager 本身不支持但 [webhook receiver](https://prometheus.io/docs/alerting/configuration/#webhook_config) 支持的通知机制集成。

* [Alertsnitch](https://gitlab.com/yakshaving.art/alertsnitch): 将告警保存到 MySQL 数据库
* [AWS SNS](https://github.com/DataReply/alertmanager-sns-forwarder)
* [DingTalk](https://github.com/timonwong/prometheus-webhook-dingtalk)
* [GELF](https://github.com/b-com-software-basis/alertmanager2gelf)
* [IRC Bot](https://github.com/multimfi/bot)
* [JIRAlert](https://github.com/free/jiralert)
* [Phabricator / Maniphest](https://github.com/knyar/phalerts)
* [prom2teams](https://github.com/idealista/prom2teams): 转发通知给微软团队
* [Rocket.Chat](https://rocket.chat/docs/administrator-guides/integrations/prometheus/)
* [ServiceNow](https://github.com/FXinnovation/alertmanager-webhook-servicenow)
* [SMS](https://github.com/messagebird/sachet): 支持[多种提供商](https://github.com/messagebird/sachet/blob/master/examples/config.yaml)
* [SNMP traps](https://github.com/maxwo/snmp_notifier)
* [Telegram bot](https://github.com/inCaller/prometheus_bot)
* [XMPP Bot](https://github.com/jelmer/prometheus-xmpp-alerts)
* [Zoom](https://github.com/Code2Life/nodess-apps/tree/master/src/zoom-alert-2.0)

## 管理 <a id="management"></a>

Prometheus 不包含配置管理功能，允许您将其与现有系统集成或在其之上构建。

* [Prometheus Operator](https://github.com/coreos/prometheus-operator): 在 Kubernetes 上管理 Prometheus
* [Promgen](https://github.com/line/promgen): Prometheus 和 Alertmanager 的 Web UI 和配置生成器

## 其它 <a id="other"></a>

* [karma](https://github.com/prymitive/karma): 告警仪表板
* [PushProx](https://github.com/RobustPerception/PushProx): 横向NAT和类似网络架构的代理
* [Promregator](https://github.com/promregator/promregator): 云原生应用的发现和数据采集

