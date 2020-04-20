---
title: 告警概述
---

# 告警概述

Prometheus 的告警分为两个部分。Prometheus服务中的告警规则将告警发送到 Alertmanager。然后，[Alertmanager](https://prometheus.io/docs/alerting/alertmanager) 通过电子邮件，通知系统和聊天平台等方法管理这些告警，包括静默，禁止，汇总和发出通知。

设置告警和通知的主要步骤是:

* 设置和[配置](https://prometheus.io/docs/alerting/configuration) Alertmanager
* [配置 Prometheus](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#alertmanager_config) 与 Alertmanagert 通信
* 在Prometheus中创建[告警规则](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)

