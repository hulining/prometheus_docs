---
title: 联合
---

# 联合

联合允许 Prometheus 服务从另一台 Prometheus 服务采集指定的时间序列。

## 用例 <a id="use-cases"></a>

下面有不同的联合用例。通常，它用于实现可扩展的 Prometheus 监控设置或将相关数据指标从一个 Prometheus 服务引入到另一项服务。

### 分层的联合 <a id="hierarchical-federation"></a>

分层联合使 Prometheus 可以扩展到具有数十个数据中心和数百万个节点的环境。在此用例中，联合拓扑类似于一棵树，更高级别的 Prometheus 服务从大量从属 Prometheus 服务收集聚合的时间序列数据。

例如，一种架构可能由每个数据中心的 Prometheus 服务收集详细信息的数据\(向下展开为实例级别\)，以及一组全局 Prometheus 服务，它们仅从这些本地服务收集和存储聚合数据\(向下展开为作业级别\)。这提供了全局汇总视图和局部详细视图。

### 跨服务的联合 <a id="cross-service-federation"></a>

在跨服务联合中，一个 Prometheus 服务配置为从另一个 Prometheus 服务中提取所选数据，以便对单个服务中的两个数据集启用警报和查询。

例如，运行多个服务的集群调度程序可能会暴露有关在集群上运行的服务实例的资源使用情况信息\(如内存和CPU使用情况\)。另一方面，在该集群上运行的服务仅暴露特定于应用程序的服务数据指标。通常，这两组指标都是由单独的 Prometheus 服务采集的。使用联合，包含服务级别数据指标的 Prometheus 服务可以从群集 Prometheus 中提取有关其特定服务的群集资源使用情况数据指标，以便可以在该服务器中使用这两组数据指标。

## 配置联合 <a id="configuring-federation"></a>

在 Prometheus 服务上，任何`/federate`端点都可以为该服务器中选定的时间序列检索当前值。必须至少指定一个`match[]` URL 参数以选择要暴露的序列。每个`match[]`参数都需要指定一个如`up`或`{job="api-server"}`的[即时向量选择器](basics.md#instant-vector-selectors)。如果指定多个`match[]`参数，则将选择所有匹配序列的并集。

要将数据指标从一台服务联合到另一台服务器，请将目标P rometheus 服务配置为从源服务的`/federate`端点进行采集，同时还启用`honor_labels`采集选项\(不覆盖源服务器暴露的任何标签\)并传递所需的`match[]`参数。例如，以下`scrape_configs`将带有标签`job="prometheus"`或以`job`开头的数据指标名称的任何数据序列从`source-prometheus-{1,2,3}:9090` Prometheus 服务联合到采集 Prometheus：

```text
scrape_configs:
  - job_name: 'federate'
    scrape_interval: 15s

    honor_labels: true
    metrics_path: '/federate'

    params:
      'match[]':
        - '{job="prometheus"}'
        - '{__name__=~"job:.*"}'

    static_configs:
      - targets:
        - 'source-prometheus-1:9090'
        - 'source-prometheus-2:9090'
        - 'source-prometheus-3:9090'
```

