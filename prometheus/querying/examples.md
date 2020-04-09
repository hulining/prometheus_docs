---
title: 查询示例
---

# 查询示例

## 简单的时间序列选择器

返回数据指标名称为`http_requests_total`的所有时间序列：

```text
http_requests_total
```

返回数据指标名称为`http_requests_total`及给定`job`和`handler`标签的所有时间序列：

```text
http_requests_total{job="apiserver", handler="/api/comments"}
```

返回相同向量的整个时间范围\(在本例中为5分钟\)，使其成为范围向量：

```text
http_requests_total{job="apiserver", handler="/api/comments"}[5m]
```

请注意，无法直接绘制范围向量的表达式结果，而是在表达式浏览器的表格视图\("控制台"\)中查看。

使用正则表达式，您可以只选择`job`标签值与特定模式匹配的时间序列，在本例子中，所有`job`标签以`server`结尾的向量：

```text
http_requests_total{job=~".*server"}
```

Prometheus中的所有正则表达式都使用[RE2语法](https://github.com/google/re2/wiki/Syntax)。

想要选择除了 4xx 以外的所有 HTTP 状态码，你可以执行：

```text
http_requests_total{status!~"4.."}
```

## 子查询

返回过去 30 分钟的 http\_requests\_total 指标的 5 分钟内的平均速率，分辨率为 1 分钟.

```text
rate(http_requests_total[5m])[30m:1m]
```

这是一个嵌套子查询的示例。`deriv`函数的子查询使用默认分辨率。请注意，不必要地使用子查询是不明智的。

```text
max_over_time(deriv(rate(distance_covered_total[5s])[30s:5s])[10m:])
```

## 使用函数、操作符等

返回最近 5 分钟内指标名称为`http_requests_total`的所有时间序列的每秒速率：

```text
rate(http_requests_total[5m])
```

假设`http_requests_total`时间序列都有`job`\(按作业名称进行划分\)和`instance`\(按作业的实例进行划分\)，我们可能希望得到的输出时间序列较少，所以对所有实例的速率进行求和，但仍然保留`job`维度：

```text
sum by (job) (
  rate(http_requests_total[5m])
)
```

如果我们有具有两个相同维度标签的不同数据指标，则可以对它们进行二元运算，并且具有相同标签集的两侧的元素都将匹配并传播到输出。例如：此实例表达式为每个实例以 MiB 返回未使用的内存\(在虚构的集群上暴露了有关其运行实例的这些指标\)：

```text
(instance_memory_limit_bytes - instance_memory_usage_bytes) / 1024 / 1024
```

相同的表达式，但根据应用汇总，可以这么写：

```text
sum by (app, proc) (
  instance_memory_limit_bytes - instance_memory_usage_bytes
) / 1024 / 1024
```

如果相同的虚拟集群调度程序对每个实例暴露了以下 CPU 使用率指标：

```text
instance_cpu_time_ns{app="lion", proc="web", rev="34d0f99", env="prod", job="cluster-manager"}
instance_cpu_time_ns{app="elephant", proc="worker", rev="34d0f99", env="prod", job="cluster-manager"}
instance_cpu_time_ns{app="turtle", proc="api", rev="4d3a513", env="prod", job="cluster-manager"}
instance_cpu_time_ns{app="fox", proc="widget", rev="4d3a513", env="prod", job="cluster-manager"}
```

我们可以按应用程序\(`app`\)和进程类型\(`proc`\)分组，排名前 3 位的 CPU 用户是这样的：

```text
topk(3, sum by (app, proc) (rate(instance_cpu_time_ns[5m])))
```

假设此指标每个运行实例包含一个时间序列，则可以像这样计算每个应用程序的运行实例数：

```text
count by (app) (instance_cpu_time_ns)
```

