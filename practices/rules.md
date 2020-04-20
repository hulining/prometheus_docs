---
title: 记录规则
sort_rank: 6
---

# 记录规则

[记录规则](../prometheus/configuration/recording_rules.md)的一致命名方案使一目了然地解释规则的含义变得更加容易。通过使错误或无意义的计算脱颖而出，还可以避免错误。

此页面记录了如何正确进行汇总并提出了命名约定。

## 命名和汇总 <a id="naming-and-aggregation"></a>

记录规则应为一般形式 `level:metric:operations`。`level` 代表规则输出的聚合级别和标签。`metric` 是度量名称，使用`rate()` 或 `irate()`时，除了从计数器剥离 `_total`，应保持不变。`_total` off counters when using `rate()` or `irate()`. `operations` 是应用到度量标准的操作列表，最新操作优先。

保持数据指标名称不变，可以轻松了解一个数据指标是什么，也可以在代码库中轻松找到。

为了保持操作整洁,如果还有其他操作，则将`_sum`省略，如`sum()`。关联操作可以合并\(如，`min_min` 与 `min` 相同\).

如果没有明显的操作要使用，请使用 `sum`。通过除法计算比率时，请使用`_per_`分隔度数据指标并将操作叫做`ratio`。

汇总比率时，分别汇总分子和分母，然后相除。不要取比率的平均值或平均值的平均值，因为这在统计上是无效的。

当汇总 Summary 的`_count` 和 `_sum` 并相除计算平均观察值时，将其视为比率是不实用的。而是保留不带 `_count` 或 `_sum` 后缀的数据指标，并将操作中的`rate`替换为`mean`。这表示该时间段内的平均观测大小。

始终使用要聚合的标签指定一个`without`子句。这是为了保留所有其他标签，例如`job`，这将避免冲突并为您提供更多有用的指标和警报。

## 示例 <a id="examples"></a>

_注意在两个向量之间的行上有缩进运算符的缩进样式。为了在 Yaml 中使这种样式，使用_[_带有缩进指示符的块引号_](https://yaml.org/spec/1.2/spec.html#style/block/scalar)_\(例如`| 2`\)._

汇总具有`path`标签的请求速率:

```text
- record: instance_path:requests:rate5m
  expr: rate(requests_total{job="myjob"}[5m])

- record: path:requests:rate5m
  expr: sum without (instance)(instance_path:requests:rate5m{job="myjob"})
```

计算请求失败率并汇总到作业级别失败率:

```text
- record: instance_path:request_failures:rate5m
  expr: rate(request_failures_total{job="myjob"}[5m])

- record: instance_path:request_failures_per_requests:ratio_rate5m
  expr: |2
      instance_path:request_failures:rate5m{job="myjob"}
    /
      instance_path:requests:rate5m{job="myjob"}

# Aggregate up numerator and denominator, then divide to get path-level ratio.
- record: path:request_failures_per_requests:ratio_rate5m
  expr: |2
      sum without (instance)(instance_path:request_failures:rate5m{job="myjob"})
    /
      sum without (instance)(instance_path:requests:rate5m{job="myjob"})

# No labels left from instrumentation or distinguishing instances,
# so we use 'job' as the level.
- record: job:request_failures_per_requests:ratio_rate5m
  expr: |2
      sum without (instance, path)(instance_path:request_failures:rate5m{job="myjob"})
    /
      sum without (instance, path)(instance_path:requests:rate5m{job="myjob"})
```

根据一个 Summary 类型的数据指标计算一段时间内的平均延迟:

```text
- record: instance_path:request_latency_seconds_count:rate5m
  expr: rate(request_latency_seconds_count{job="myjob"}[5m])

- record: instance_path:request_latency_seconds_sum:rate5m
  expr: rate(request_latency_seconds_sum{job="myjob"}[5m])

- record: instance_path:request_latency_seconds:mean5m
  expr: |2
      instance_path:request_latency_seconds_sum:rate5m{job="myjob"}
    /
      instance_path:request_latency_seconds_count:rate5m{job="myjob"}

# Aggregate up numerator and denominator, then divide.
- record: path:request_latency_seconds:mean5m
  expr: |2
      sum without (instance)(instance_path:request_latency_seconds_sum:rate5m{job="myjob"})
    /
      sum without (instance)(instance_path:request_latency_seconds_count:rate5m{job="myjob"})
```

使用`avg()` 函数计算出不同 instance 和 path 的平均查询率:

```text
- record: job:request_latency_seconds_count:avg_rate5m
  expr: avg without (instance, path)(instance:request_latency_seconds_count:rate5m{job="myjob"})
```

请注意，在进行聚合时，与输入数据指标名称相比，将`without`子句中的标签从输出数据指标名称的级别中删除。没有聚合时，级别始终匹配。如果不是这种情况，则可能是规则中有错误。

