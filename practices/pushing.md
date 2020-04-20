---
title: 什么时候使用 Pushgateway
sort_rank: 7
---

# 什么时候使用 Pushgateway

Pushgateway 是一种可让您从无法数据采集的作业中推送指标的中间服务。有关详细信息，请参见[推送指标](../instrumenting/pushing.md)。

## 我应该使用 Pushgateway 吗? <a id="should-i-be-using-the-pushgateway"></a>

**我们仅建议在某些特定情况下使用 Pushgateway。** 盲目使用 Pushgateway 而不使用 Prometheus 常用的拉取数据模型进行常规指标收集时会遇到一些陷阱:

* 通过单个 Pushgateway 监控多个实例时，Pushgateway 既会成为单个故障点，也可能成为瓶颈。
* 会丢失 Prometheus 通过`up`数据指标\(在每个采集目标上\)生成的自动实例运行状况监控
* 除非通过 Pushgateway 的 API 手动删除了这些序列，否则 Pushgateway 永远不会忽略向其推送的系列，并将它们永久暴露给 Prometheus。

当一个作业的多个实例通过实例标签或类似标签在 Pushgateway 中区分其指标时，后一点特别重要。即使重命名或删除了原始实例，该实例的指标也将保留在 Pushgateway 中。这是因为作为指标缓存的 Pushgateway 的生命周期与将指标推送到它的流程的生命周期从根本上是分开的。将此与 Prometheus 常规的拉取数据监控进行对比: 实例消失\(有意无意\)时，其指标将随之自动消失。使用 Pushgateway 时，情况并非如此，您必须手动删除所有过时的指标，或者自动执行生命周期同步。

**通常，Pushgateway 的唯一有效用例是接收服务级别批处理作业的结果**。"服务级别" 的批处理作业是与特定机器或作业实例在语义上不相关的作业\(例如，删除整个服务的多个用户的批处理作业\)。此类作业的数据指标不应包含机器或实例标签，以使特定机器或实例的生命周期与推送的指标解耦。这减轻了管理 Pushgateway 中过量数据指标的负担。另请参阅[监控批处理作业的最佳做法](instrumentation.md#batch-jobs)

## 替代策略 <a id="alternative-strategies"></a>

如果入站防火墙或 NAT 阻止您从目标中提取指标，请考虑将 Prometheus 服务也移到网络防火墙后面。我们通常建议在与受监视实例相同的网络上运行 Prometheus 服务。否则，请考虑使用 [PushProx](https://github.com/RobustPerception/PushProx)，它可以让 Prometheus 穿越防火墙或 NAT。

对于与机器相关的批处理作业\(如自动安全更新定时任务或运行的配置管理客户端\)，请使用 [Node Exporter](https://github.com/prometheus/node_exporter) 的 [textfile collector](https://github.com/prometheus/node_exporter#textfile-collector) 而不是 Pushgateway 公开的结果指标。

