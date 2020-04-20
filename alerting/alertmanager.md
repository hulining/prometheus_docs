---
title: Alertmanager
---

# Alertmanager

[Alertmanager](https://github.com/prometheus/alertmanager) 处理由客户端应用程序\(如 Prometheus 服务\)发送的警报。它负责将重复数据删除，分组和路由到正确的接收组件集成，如电子邮件、PagerDuty 或 OpsGenie。 它还负责沉默和抑制告警。

下面描述了 Alertmanager 实现的核心概念。请查阅[配置文档](configuration.md)以了解如何更详细地使用它们。

## 告警分组 <a id="grouping"></a>

分组将类似性质的告警分类为单个通知。在较大的架构中，许多系统一旦出现故障可能同时触发成百上千的告警时特别有用

**示例**: 当网络进行分区时，集群中正在运行数十个或数百个服务实例。您有一半的服务实例不再可以访问数据库。Prometheus 中的告警规则配置为在每个服务实例无法与数据库通信时为其发送警报。其结果就是数百个告警被发送到 Alertmanager。

作为用户，人们只希望获得一个告警，同时仍然能够准确查看受影响的服务实例。因此，可以将 Alertmanager 配置为按告警的集群和告警的名称分类告警，以便发送一个简洁的通知。

告警的分组，分组通知的时间以及这些通知的接收者由配置文件中的路由树配置。

## 告警抑制 <a id="inhibition"></a>

抑制是一种概念，如果某些其他告警已经触发，则抑制某些告警的通知。

**示例**: 假设有一个通知您无法访问整个集群的告警。如果该特定警报正在触发，可以将 Alertmanager 配置为使与该群集有关的所有其他告警静音。这样可以防止与实际问题无关的数百或数千个触发告警通知。

通过 Alertmanager 的配置文件配置抑制告警规则。

## 告警静默 <a id="silences"></a>

静默是一种可以在给定时间内直接使告警静音的方法。静默是根据匹配器配置的，就像路由树一样。检查传入告警是否与活动静默等值或正则表达式匹配。如果匹配，则不会针对该告警发送任何通知。

在Alertmanager 的 Web 页面中配置沉默规则。

## 客户端行为 <a id="client-behavior"></a>

Alertmanager 对客户端行为有[特殊要求](clients.md)。这些仅与不使用 Prometheus 发送警报的高级用例有关。

## 高可用 <a id="high-availability"></a>

Alertmanager 支持配置创建高可用性集群。可以使用`--cluster-*`标志进行配置。

重要的是不要平衡 Prometheus 及 Alertmanagers 之间的流量，而是将 Prometheus 指向所有 Alertmanagers 的列表。

