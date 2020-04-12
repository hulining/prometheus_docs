---
title: 函数
---

# 函数

某些函数具有默认参数，例如`year(v=vector(time()) instant-vector)`。这意味着参数`v`是一个即时向量，如果不提供，它将默认为表达式`vector(time())`的值。

## abs\(\)

`abs(v instant-vector)`返回所有样本值均转换为绝对值的输入向量。

## absent\(\)

如果传递给它的向量具有任何元素，`absent(v instant-vector)`返回一个空向量。如果传递给它的向量没有元素，则返回值为 1 的一元素向量。

这对于在给定数据指标名称和标签组合的时间序列不存在时发出警报非常有用。

```text
absent(nonexistent{job="myjob"})
# => {job="myjob"}

absent(nonexistent{job="myjob",instance=~".*"})
# => {job="myjob"}

absent(sum(nonexistent{job="myjob"}))
# => {}
```

在前两个示例中，`absent()`试图从输入向量中导出一元素输出向量。

## absent\_over\_time\(\)

如果传递给它的范围向量有任何元素，`absent_over_time(v range-vector)`返回一个空向量。如果传递给它的范围向量没有元素，则返回值为 1 的一元素向量。

这对于在给定数据指标名称和标签组合的时间序列在一定时间内不存在时发出警报非常有用。

```text
absent_over_time(nonexistent{job="myjob"}[1h])
# => {job="myjob"}

absent_over_time(nonexistent{job="myjob",instance=~".*"}[1h])
# => {job="myjob"}

absent_over_time(sum(nonexistent{job="myjob"})[1h:])
# => {}
```

在前两个示例中，`absent_over_time()`试图从输入向量中导出一元素输出向量。

## ceil\(\)

`ceil(v instant-vector)`将`v`中所有元素的样本值增加到最接近的整数。

## changes\(\)

对于每个输入时间序列，`changes(v range-vector)`将其值在提供的时间范围内变化的次数作为即时向量返回。

## clamp\_max\(\)

`clamp_max(v instant-vector, max scalar)`将`v`中所有元素的样本值设置一个最大上限。

## clamp\_min\(\)

`clamp_min(v instant-vector, min scalar)`将`v`中所有元素的样本值设置一个最小下限。

## day\_of\_month\(\)

`day_of_month(v=vector(time()) instant-vector)`返回 UTC 时间中每个给定时间的是当月的第几天。返回值是 1-31。

## day\_of\_week\(\)

`day_of_week(v=vector(time()) instant-vector)`返回 UTC 时间中每个给定时间是当周的第几天。返回值是 0-6。其中 0 表示星期日。

## days\_in\_month\(\)

`days_in_month(v=vector(time()) instant-vector)`返回 UTC 时间中每个给定时间的月份有几天。返回值是 28-31。

## delta\(\)

`delta(v range-vector)`计算范围向量`v`中每个时间序列元素的第一个值和最后一个值之间的差，返回带有和给定差异项等效标签的即时向量。根据范围矢量选择器中的指定，可以推断整个时间范围的增量，因此即使采样值都是整数，也可以得到非整数结果。

以下示例表达式返回现在和 2 小时前的 CPU 温度差异：

```text
delta(cpu_temp_celsius{host="zeus"}[2h])
```

`delta`应该与 gauges 一起使用。

## deriv\(\)

`deriv(v range-vector)`使用[简单线性回归](https://en.wikipedia.org/wiki/Simple_linear_regression)来计算范围向量`v`中时间序列的每秒导数。

`deriv`应该与 gauges 一起使用。

## exp\(\)

`exp(v instant-vector)`计算`v`中所有元素的指数函数。特殊情况是：

* `Exp(+Inf) = +Inf`
* `Exp(NaN) = NaN`

## floor\(\)

`floor(v instant-vector)`将`v`中所有元素的样本值向下舍入到最近的整数。

## histogram\_quantile\(\)

`histogram_quantile(φ float, b instant-vector)`从[直方图](metric_types.md#histogram)的`b`区间向量计算 φ-quantile\(0 ≤ φ ≤ 1\)分位数。\(有关 φ-quantile 分位数的详细说明以及基本使用直方图指标类型的信息，请参见[histograms and summaries](histograms.md)\)。`b`中的样本是每个区间中观察值的计数。每个样本必须具有标签`le`，其中标签值表示区间上限\(没有此类标签的样本将被忽略\)。[直方图数据指标类型](metric_types.md#histogram)会自动提供带有`_bucket`后缀和适当标签的时间序列。

使用`rate()`函数指定分位数计算的时间窗口。

示例：一个直方图指标名称为`http_request_duration_seconds`。要计算最近 10m 的请求持续时间的 90%，请使用以下表达式：

```text
histogram_quantile(0.9, rate(http_request_duration_seconds_bucket[10m]))
```

对于`http_request_duration_seconds`中的每个标签组合计算分位数。要进行汇总，请在`rate()`函数外使用`sum()`。由于`histogram_quantile()`要求使用`le`标签，因此必须将其包含在`by`子句中。以下表达式按`job`汇总了 90%：

```text
histogram_quantile(0.9, sum(rate(http_request_duration_seconds_bucket[10m])) by (job, le))
```

要汇总所有内容，仅指定`le`标签：

```text
histogram_quantile(0.9, sum(rate(http_request_duration_seconds_bucket[10m])) by (le))
```

`histogram_quantile()`函数通过假设区间内的线性分布来对分位数进行插值。存储桶的最高上限必须为`+Inf`\(否则返回`NaN`\)。如果分位数位于最高的区间中，则返回第二高的区间的上限。如果该区间的上限大于0，则将最低区间的下限假定为0。在这种情况下，通常在该区间中应用线性插值。否则，将为位于最低区间中的分位数返回最低区间的上限。

如果`b`包含少于两个区间，则返回`NaN`。如果`φ<0`，则返回`-Inf`。 对于`φ> 1`，返回`+Inf`。

## holt\_winters\(\)

`holt_winters(v range-vector, sf scalar, tf scalar)`根据`v`中范围向量生成时间序列的平滑值。平滑因子`sf`越低，则对旧数据的重视程度越高。趋势因子`tf`越高，就越多考虑数据的趋势。`sf`和`tf`都必须介于0和1之间。

`holt_winters`应该与 gauges 一起使用。

## hour\(\)

`hour(v=vector(time()) instant-vector)`返回 UTC 时间中每个给定时间是一天中第几小时。 返回值是从0到23。

## idelta\(\)

`idelta(v range-vector)`计算范围向量`v`中最后两个样本之间的差，返回具有和给定差异项等效标签的即时向量。

`idelta`应该与 gauges 一起使用。

## increase\(\)

`increase(v range-vector)`计算范围向量中时间序列的增长。单调性中断\(例如由于目标重新启动而导致的计数器重置\)会自动进行调整。根据范围向量选择器中的指定，可以推断出覆盖整个时间范围的增长。因此，即使计数器仅以整数增量增加，也可能会获得非整数结果。

以下示例表达式返回范围向量中每个时间序列在最近 5 分钟内测得的 HTTP 请求数：

```text
increase(http_requests_total{job="api-server"}[5m])
```

`increase()`只能与 counter 一起使用。它是`rate(v)`乘以指定时间范围窗口内的秒数的语法糖，主要用于可读性。在记录规则中使用`rate`函数，以便在每秒的基础上持续跟踪增长情况。

## irate\(\)

`irate(v range-vector)`计算范围矢量中时间序列的每秒瞬时增加率。这基于最后两个数据点。单调性中断\(例如由于目标重新启动而导致的计数器重置\)会自动进行调整。

以下示例表达式返回范围向量中每个时间序列的两个最近数据点的 HTTP 请求的每秒速率，该速率最多可向后查询 5分钟：

```text
irate(http_requests_total{job="api-server"}[5m])
```

仅在绘制易变、快速变化的计数器时才使用`irate`函数。告警和缓慢移动计数器使用`rate`函数，因为速率的细微改变会在 For 子句中重置，且完全没有峰值的图形很难读取。

请注意，将`irate()`与聚合运算\(如`sum()`\)或随时间进行聚合的函数\(任何以`_over_time`结尾的函数\)结合使用时，请先获取`irate()`，然后进行聚合。否则，当目标重新启动时，`irate()`无法检测到计数器重置。

## label\_join\(\)

对于`v`中的每个时间序列，`label_join(v instant-vector, dst_label string, separator string, src_label_1 string, src_label_2 string, ...)`使用`separator`将所有的`src_labels`的所有值连接在一起，并返回带有包含已连接的`dst_label`标签的时间序列。此函数中可以有任意数量的`src_labels`。

该示例将返回每个时间序列都带有一个值为`a,b,c`的`foo`标签的新向量：

```text
label_join(up{job="api-server",src1="a",src2="b",src3="c"}, "foo", ",", "src1", "src2", "src3")
```

## label\_replace\(\)

对于`v`中的每个时间序列，`label_replace(v instant-vector, dst_label string, replacement string, src_label string, regex string)`将正则表达式`regex`与标签`src_label`匹配。如果匹配，则返回时间序列，其中标签`dst_label`被替换为`replacement`。 `$1`表示被第一个匹配的子组替换，`$2`表示被第二个匹配的子组替换，依次类推。如果正则表达式不匹配，则时间序列将保持不变。

该示例将返回每个时间序列都带有一个值为`a`的`foo`标签的新向量：

```text
label_replace(up{job="api-server",service="a:c"}, "foo", "$1", "service", "(.*):.*")
```

## ln\(\)

`ln(v instant-vector)`计算`v`中所有元素的自然对数。特殊情况如下：

* `ln(+Inf) = +Inf`
* `ln(0) = -Inf`
* `ln(x < 0) = NaN`
* `ln(NaN) = NaN`

## log2\(\)

`log2(v instant-vector)`计算`v`中所有元素的二进制对数。特殊情况与`ln`函数相同。

## log10\(\)

log10\(v instant-vector\)计算`v`中所有元素的十进制对数。特殊情况与`ln`函数相同。

## minute\(\)

`minute(v=vector(time()) instant-vector)`返回 UTC 时间中每个给定时间是一个小时中第几分钟。返回值是 0-59。

## month\(\)

`month(v=vector(time()) instant-vector)`返回 UTC 时间中每个给定时间是一年中第几月。返回值是 0-12。

## predict\_linear\(\)

`predict_linear(v range-vector, t scalar)`使用[简单线性回归](https://en.wikipedia.org/wiki/Simple_linear_regression)，基于范围向量`v`预测从现在开始的时间序列`t`秒的值。

`predict_linear`应该与 gauges 一起使用。

## rate\(\)

`rate(v range-vector)`计算范围向量中时间序列的每秒平均增长率。 单调性中断\(例如由于目标重新启动而导致的计数器重置\)会自动进行调整。而且，计算会推断到时间范围的末尾，从而允许遗漏采集数据或采集周期与该范围的时间段不完全对齐。

以下示例表达式返回范围向量中每个时间序列在过去 5 分钟内每秒的 HTTP 请求速率：

```text
rate(http_requests_total{job="api-server"}[5m])
```

`rate`只能与 counter 一起使用。它最适合于警报和慢速计数器的图形显示。

请注意，在将`rate()`与聚合运算\(例如`sum()`\)或随时间进行聚合的函数\(任何以`_over_time`结尾的函数\)结合使用时，请始终先获取`rate()`，然后再进行聚合。否则，当目标重新启动时，`rate()`无法检测到计数器重置。

## resets\(\)

对于每个输入时间序列，`resets(v range-vector)`将提供的时间范围内的计数器重置次数作为即时向量返回。两个连续采样值的任何减少都将解释为计数器复位。

`resets`只能与 counter 一起使用。

## round\(\)

`round(v instant-vector, to_nearest=1 scalar)`将`v`中所有元素的样本值四舍五入到最接近的整数。通过四舍五入解决。可选的`to_nearest`参数允许指定样本值应四舍五入到的最接近倍数。该倍数也可以是分数。

## scalar\(\)

给定一个单元素输入向量，`scalar(v instant-vector)`返回该单个元素的样本值作为标量。如果输入向量不完全具有一个元素，则标量将返回`NaN`。

## sort\(\)

`sort(v instant-vector)`返回按其样本值升序排列的向量元素。

## sort\_desc\(\)

与`sort`相同，但按降序排序。

## sqrt\(\)

`sqrt(v instant-vector)`计算v中所有元素的平方根

## time\(\)

`time()`返回自 UTC 1970-01-01 以来的秒数。请注意，这实际上并不返回当前时间，而是返回要计算的表达式的时间。

## timestamp\(\)

`timestamp(v instant-vector)`返回自 UTC 1970-01-01以来，给定矢量的每个样本的时间戳。

_该函数在 Prometheus 2.0 中添加_

## vector\(\)

`vector(s scalar)`将标量`s`作为没有标签的向量返回

## year\(\)

`year(v=vector(time()) instant-vector)`返回 UTC 时间中每个给定时间的年份。

## &lt;aggregation&gt;\_over\_time\(\) <a id="aggregation_over_time"></a>

以下函数允许随着时间的推移聚合给定范围向量的每个序列，并返回具有每个序列聚合结果的即时向量：

* `avg_over_time(range-vector)`: 指定间隔中所有采样点的平均值
* `min_over_time(range-vector)`: 指定间隔中所有采样点的最小值
* `max_over_time(range-vector)`: 指定间隔中所有采样点的最大值
* `sum_over_time(range-vector)`: 指定时间间隔内所有采样点值的和
* `count_over_time(range-vector)`: 指定间隔内所有值的计数
* `quantile_over_time(scalar, range-vector)`: 指定间隔中值的 φ-quantile 分位数\(0≤φ≤1\)
* `stddev_over_time(range-vector)`: 在指定间隔内值的总体标准差
* `stdvar_over_time(range-vector)`: 在指定间隔内值的总体标准方差

请注意，即使在整个时间间隔内这些值的间隔不相等，指定时间间隔内的所有值在聚合中的权重也相同。

