# Table of contents

* [Prometheus 中文文档](README.md)

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

## instrumenting

* [客户端库](instrumenting/clientlibs.md)
* [编写客户端库](instrumenting/writing_clientlibs.md)
* [推送数据指标](instrumenting/pushing.md)
* [数据导出及相关整合](instrumenting/exporters.md)
* [编写数据导出器](instrumenting/writing_exporters.md)
* [公开的格式](instrumenting/exposition_formats.md)

## operating

* [安全](operating/security.md)
* [集成](operating/integrations.md)

## alerting

* [overview](alerting/overview.md)
* [alertmanager](alerting/alertmanager.md)
* [configuration](alerting/configuration.md)
* [clients](alerting/clients.md)
* [notifications](alerting/notifications.md)
* [notification\_examples](alerting/notification_examples.md)
* [management\_api](alerting/management_api.md)

## practices

* [naming](practices/naming.md)
* [instrumentation](practices/instrumentation.md)
* [consoles](practices/consoles.md)
* [histograms](practices/histograms.md)
* [alerting](practices/alerting.md)
* [rules](practices/rules.md)
* [pushing](practices/pushing.md)
* [remote\_write](practices/remote_write.md)

## guides

* [multi-target-exporter](guides/multi-target-exporter.md)
* [file-sd](guides/file-sd.md)
* [go-application](guides/go-application.md)
* [cadvisor](guides/cadvisor.md)
* [node-exporter](guides/node-exporter.md)
* [basic-auth](guides/basic-auth.md)
* [query-log](guides/query-log.md)
* [tls-encryption](guides/tls-encryption.md)

