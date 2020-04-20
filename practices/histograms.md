---
title: histogram and summary
sort_rank: 4
---

# Histogram and Summary

histogram 和 summary 是更复杂的指标类型。单个 histogram 或 summary 不仅会创建多个时间序列，而且正确使用这些指标类型也更加困难。本节帮助您在使用时选择和配置适当的数据指标类型。

## 库支持 <a id="library-support"></a>

首先，检查库对 [histogram](/concepts/metric_types/#histogram)和[summary](/concepts/metric_types/#summary)的支持。

一些库仅支持两种类型之一，或者它们仅以有限的方式支持 summary\(缺少 [分位数计算]()\).

## 观测计数和求和 <a id="count-and-sum-of-observations"></a>

histogram 和 summary 都是样本观察值，通常是请求持续时间或响应大小。它们跟踪观测值的_数量_和观测值的_总和_，从而使您可以计算观测值的_平均值_。请注意，观测值的数量\(在 Prometheus 中显示为带有`_count`后缀的时间序列\)本质上是一个计数器\(如上所述，它只会上升\)。观测值的总和\(显示为带有`_sum`后缀的时间序列\)也可以被认为是计数器，只要没有负的观测值即可。显然，请求持续时间或响应大小永远不会为负。但是原则上，您可以使用 histogram 和 summary 类型的数据指标观察负值\(例如，摄氏温度\)。在这种情况下，观察值的总和可能会下降，因此您无法再对他应用`rate()`函数。

要根据称为`http_request_duration_seconds`的 histogram 或 summary 类型的数据指标计算最近5分钟内的平均请求时长，请使用以下表达式：

```text
  rate(http_request_duration_seconds_sum[5m])
/
  rate(http_request_duration_seconds_count[5m])
```

## 应用性能指数 <a id="apdex-score"></a>

直接使用 histogram 类型\(而不是 summary 类型\)是对落入特定观察值桶中的观察值进行计数。

可能有一个 SLO 是在 300 毫秒内处理 95％ 的请求。在这种情况下，请将 histogram 类型的数据指标配置为具有 0.3 秒上限的存储桶。然后，您可以直接表示 300 毫秒内服务的请求数量比值，并在该值降至 0.95 以下时轻松发出警报。以下表达式按最近 5 分钟内服务的请求作业计算它。请求时长被保存为 histogram 类型的指标，称为 `http_request_duration_seconds`.

```text
  sum(rate(http_request_duration_seconds_bucket{le="0.3"}[5m])) by (job)
/
  sum(rate(http_request_duration_seconds_count[5m])) by (job)
```

您可以通过类似的方式来估算著名的 [Apdex 分数](http://en.wikipedia.org/wiki/Apdex)\(应用性能指数\)。

配置一个以目标请求持续时间为上限的存储桶，并以容忍的请求持续时间\(通常为目标请求持续时间的4倍\)作为另一个存储桶。例如: 目标请求持续时间为 300ms,允许的请求持续时间为 1.2s。 以下表达式产生最近 5 分钟内每个作业的 Apdex 分数：

```text
(
  sum(rate(http_request_duration_seconds_bucket{le="0.3"}[5m])) by (job)
+
  sum(rate(http_request_duration_seconds_bucket{le="1.2"}[5m])) by (job)
) / 2 / sum(rate(http_request_duration_seconds_count[5m])) by (job)
```

请注意，我们将两个存储桶的总和相除。原因是 histogram 类型的数据指标中的存储桶是[累积](https://en.wikipedia.org/wiki/Histogram#Cumulative_histogram)的。`le="0.3"`存储桶也包含在`le="1.2"`存储桶中；将其除以 2 即可解决此问题。

该计算与传统的 Apdex 分数不完全匹配，因为它包括计算中可忍受的误差。

## 分位数 <a id="quantiles"></a>

您可以使用 histogram 和 summary 来计算所谓的 φ-分位数\(0 ≤ φ ≤ 1\). φ-分位数 是在 N 个观测值中排名为 φ\* N 的观测值。例如，0.5-分位数称为中位数。0.95分位数是第95个百分点

summaries 和 histograms 之间的本质区别在于，summaries 在客户端侧计算数据流的 φ-分位数并直接公开它们，而 histograms 公开桶式观察计数，而 histogram 的桶中的分位数计算在服务器端使用 [`histogram_quantile()` 函数](prometheus/latest/querying/functions/#histogram_quantile)。

两种类型有许多不同的含义

|  | Histogram | Summary |
| :---: | :---: | :---: |
| 所需配置 | 选择适合预期范围观察值的存储桶 | 选择所需的 φ-分位数和滑动窗口。其他 φ-分位数和滑动窗口以后无法计算 |
| 客户端性能 | 观测值消耗较小，因为它们只需要增加计数器 | 由于流式分位数计算，观测值消耗较大 |
| 服务端性能 | 服务器必须计算分位数。您可以使用[记录规则](/prometheus/configuration/recording_rules/#recording-rules)临时计算是否需要太长时间\(例如在大型仪表板中\) | 低服务端消耗 |
| 时间序列数\(除了`_sum`和`_count`之外\) | 每个配置的存储桶有一个时间序列. | 每个配置的分位数有一个时间序列. |
| 分位数错误\(详情见下文\) | 误差受相关观察上存储桶宽度维度的限制. | 误差受可配置值 φ 维度的限制 |
| φ分位数和滑动时间窗口的规范 | [Prometheus 表达式](/prometheus/querying/functions/#histogram_quantile). | 由客户端预先配置. |
| 聚合 | [Prometheus 表达式](/prometheus/querying/functions/#histogram_quantile). | 一般来说，[不可汇总](http://latencytipoftheday.blogspot.de/2014/06/latencytipoftheday-you-cant-average.html). |

注意表中最后一项的重要性。让我们回到在 300ms 内处理 95% 请求的 SLO。这次，您不想显示 300ms 内已处理请求的百分比，而是显示第95个百分点，即您为 95% 的请求提供服务的请求时间。为此，您可以配置摘要为 0.95-分位数，衰减时间为 5 分钟\(举例\)，也可以配置在 300ms 标记附近添加几个存储桶的 histogram，例如 `{le="0.1"}`, `{le="0.2"}`, `{le="0.3"}` 和 `{le="0.45"}`如果您的服务使用多个实例进行复制运行，则您将从其中的每个实例收集请求持续时间，然后将所有内容汇总到整体的 95%。但是，从 summary 预计算的分位数是很少是有意义的。在这种特定情况下，分位数求平均值会产生统计上无意义的值。

```text
avg(http_request_duration_seconds{quantile="0.95"}) // BAD!
```

使用 histograms 类型数据指标，使用[`histogram_quantile()` 函数](/prometheus/querying/functions/#histogram_quantile)完全可以进行聚合 [`histogram_quantile()` 函数](/prometheus/querying/functions/#histogram_quantile).

```text
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) // GOOD.
```

此外，如果您的 SLO 发生变化并且您现在想要绘制第 90 个百分位，或者您想考虑过去的 10 分钟而不是过去的 5 分钟，则只需调整上面的表达式即可，而无需重新配置客户端。

## 分位数估算的误差 <a id="errors-of-quantile-estimation"></a>

分位数，无论是计算的客户端端还是服务器端，都是估算值。重要的是要了解估算的误差。

继续上面的 histogram 示例, 想象一下，您通常的请求时长几乎都接近 220ms，换句话说，如果您可以绘制真实的直方图，您会在 220ms 处看到非常尖锐的峰值。在上面配置的 Prometheus histogram 类型的数据指标中，几乎所有观察结果以及第 95 个百分位都将落入标有`{le="0.3"}`的存储桶中，即 200ms-300ms 的存储桶。histogram 可确保真实的第 95 个百分位数在 200ms-300ms之间。要返回单个值\(而不是间隔\)，它会应用线性插值，在这种情况下将产生 295ms。计算得出的分位数给您的印象是您接近违反 SLO，但是实际上，第95个百分位数略高于 220ms，这与您的 SLO 相当符合。

我们继续想下一个场景: 后端路由的更改为所有请求时长增加了固定的 100ms。现在，请求时长在320毫秒处急剧上升，几乎所有观察结果都会落到 300ms-450ms 的存储桶中。尽管正确值接近320ms，但第 95 个百分位数计算为 442.5ms。尽管在 SLO 之外的请求只有一小部分，但计算得出的第 95 位分位数看起来要差得多。

在以上两种情况下，summary 都可以毫无问题地计算出正确的百分位数值，至少如果它在客户端使用了适当的算法\(如 \[Go 客户端使用的\]\(\([http://www.cs.rutgers.edu/~muthu/bquant.pdf\)\)\)。但是如果您需要汇总多个实例的观测值，则无法使用](http://www.cs.rutgers.edu/~muthu/bquant.pdf%29%29%29。但是如果您需要汇总多个实例的观测值，则无法使用) summary。

幸运的是，由于您适当选择了存储桶边界，即使在这个观察值分布非常不均匀的人造示例中，histogram 也能够正确识别您是否在SLO范围之内或之外。同样，分位数的实际值越接近我们的 SLO\(或换句话说，我们实际上最感兴趣的值\)，计算出的值就越准确。

现在让我们再次修改实验。在新的设置中，请求时长的分布在 150ms 处有一个尖峰，但它不像以前那样急剧，仅占 90% 的观测值。10% 的观察结果平均分布在 150ms-450ms 之间的长尾中。通过这种分布，第 95 个分位数恰好在我们的 300ms 的 SLO 处。使用 histogram，由于第 95 个百分位数的值恰好与存储桶边界之一重合，因此计算得出的值是准确的。甚至略有不同的值也将是准确的，因为相关存储桶中\(人为\)的均匀分布正是存储区中线性插值所假定的。

summary 报告的分位数误差现在变得更加有趣。summary 中的分位数误差是在 φ 维度中配置的。我们可能配置为 0.95±0.01，即计算出的值将介于 94% 和 96% 之间。具有上述分布的第 94 位为 270ms，第 96 位为 330ms。summary 报告的第 95 个百分位数的计算值可以在 270ms-330ms 之间的任何位置，这很明显是 SLO 之内与 SLO 之外的差别。

本质内容是: 如果使用 summary，则控制 φ 维度上的误差。如果使用 histogram，则控制观测值范围的误差\(通过选择适当的存储桶布局\)。分布较宽时，φ 的微小变化会导致观测值的较大偏差。分布较集中时，较小的观测值范围覆盖较大的 φ 间隔。

两条经验法则：

1. 如果需要汇总，请选择 histograms
2. 除此以外，如果您对将要观察的值的范围和分布有所了解，请选择 histograms。无论值的范围和分布如何，如果需要准确的分位数，请选择 summary

## 如果客户端库不支持所需的指标类型，该怎么办? <a id="what-can-i-do-if-my-client-library-does-not-support-the-metric-type-i-need"></a>

自行实现它！[欢迎贡献代码](https://prometheus.io/community/)。总的来说，我们期望比 summaries 更需要 histograms。histograms 在客户端库中也更容易实现，因此，如果有疑问，我们建议先实现 histograms。

