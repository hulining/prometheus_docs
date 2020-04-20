# Prometheus 中文文档

![prometheus_logo](https://prometheus.io/assets/prometheus_logo.png)

此文档是根据 [Prometheus 官网](https://prometheus.io/docs/)翻译的非官方中文手册，旨在为自己和大家提供一个关于 Prometheus 学习的中文参考。鉴于本人能力有限，翻译或理解有错误的地方，还请大家多多包涵并帮忙修订校正，不吝感谢。


## introduction

* [概述](introduction/overview.md)
* [初识 Prometheus](introduction/first_steps.md)
* [与替代品比较](introduction/comparison.md)
* [常见问题](introduction/faq.md)
* [路线图](introduction/roadmap.md)
* [相关资源](introduction/media.md)
* [相关术语](introduction/glossary.md)

## concepts

* [数据模型](concepts/data_model.md)
* [数据指标类型](concepts/metric_types.md)
* [作业和实例](concepts/jobs_instances.md)

## prometheus

* [快速开始](prometheus/getting_started.md)
* [安装](prometheus/installation.md)
* [配置](prometheus/configuration/README.md)
  * [配置](prometheus/configuration/configuration.md)
  * [定义记录规则](prometheus/configuration/recording_rules.md)
  * [告警规则](prometheus/configuration/alerting_rules.md)
  * [模板示例](prometheus/configuration/template_examples.md)
  * [模板参考](prometheus/configuration/template_reference.md)
  * [规则的单元测试](prometheus/configuration/unit_testing_rules.md)
* [查询](prometheus/querying/README.md)
  * [Prometheus  查询](prometheus/querying/basics.md)
  * [运算符](prometheus/querying/operators.md)
  * [函数](prometheus/querying/functions.md)
  * [查询示例](prometheus/querying/examples.md)
  * [HTTP API](prometheus/querying/api.md)
* [存储](prometheus/storage.md)
* [联合](prometheus/federation.md)
* [管理 API](prometheus/management_api.md)
* [Prometheus 2.0 迁移指南](prometheus/migration.md)
* [API 稳定性保证](prometheus/stability.md)

## visualization

* [表达式浏览器](visualization/browser.md)
* [Grafana 对 Prometheus 的支持](visualization/grafana.md)
* [控制台模板](visualization/consoles.md)

## operating

* [安全模型](operating/security.md)
* [集成](operating/integrations.md)

## instrumenting

* [客户端库](instrumenting/clientlibs.md)
* [编写客户端库](instrumenting/writing_clientlibs.md)
* [推送数据指标](instrumenting/pushing.md)
* [数据导出及相关集成](instrumenting/exporters.md)
* [编写数据导出器](instrumenting/writing_exporters.md)
* [公开的格式](instrumenting/exposition_formats.md)

## alerting

* [告警概述](alerting/overview.md)
* [Alertmanager](alerting/alertmanager.md)
* [配置](alerting/configuration.md)
* [发送告警](alerting/clients.md)
* [通知模板参考](alerting/notifications.md)
* [通知模板示例](alerting/notification_examples.md)
* [管理 API](alerting/management_api.md)

## practices

* [指标和标签命名](practices/naming.md)
* [控制台和仪表盘](practices/consoles.md)
* [工具](practices/instrumentation.md)
* [Histogram and Summary](practices/histograms.md)
* [告警](practices/alerting.md)
* [记录规则](practices/rules.md)
* [什么时候使用 Pushgateway](practices/pushing.md)
* [远程写调试](practices/remote_write.md)

## guides

* [使用 cAdvisor 监控 docker 容器数据指标](guides/cadvisor.md)
* [使用基于文件的服务发现来发现数据采集目标](guides/file-sd.md)
* [实现一个 Go 应用](guides/go-application.md)
* [使用 Node Exporter 监控 Linux 主机指标](guides/node-exporter.md)
* [使用基本身份验证保护 Prometheus API 和 UI 端点](guides/basic-auth.md)
* [理解并使用 multi-target exporters 模式](guides/multi-target-exporter.md)
* [使用 TLS 加密 Prometheus API 和 UI 端点](guides/tls-encryption.md)
* [使用 Prometheus 查询日志](guides/query-log.md)


