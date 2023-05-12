# 数据指标类型

Prometheus 客户端库提供了四种核心数据指标类型。这些仅在客户端库(针对特定类型的使用量身定制的API)和~~有线协议~~中有所区别。Prometheus 服务尚未使用数据类型，而是将所有数据平铺为没有类型的时间序列。将来可能会改变。

## Counter 计数器类型 <a href="#counter" id="counter"></a>

\_counter\_是一个累计类型的数据指标，它代表单调递增的计数器，其值只能在重新启动时增加或重置为 0。例如，您可以使用计数器来表示已响应的请求数，已完成或出错的任务数。

不要使用计数器来显示可以减小的值。例如，请不要使用计数器表示当前正在运行的进程数；使用 gauge 代替。

计数器的客户端库使用文档：

* [Go](https://godoc.org/github.com/prometheus/client\_golang/prometheus#Counter)
* [Java](https://github.com/prometheus/client\_java#counter)
* [Python](https://github.com/prometheus/client\_python#counter)
* [Ruby](https://github.com/prometheus/client\_ruby#counter)

## Gauge 数据轨迹类型 <a href="#gauge" id="gauge"></a>

_gauge_ 是可以任意上下波动数值的指标类型。

Gauge 通常用于测量值，例如温度或当前的内存使用量，还可用于可能上下波动的"计数"，例如请求并发数。

Gauge 的客户端库使用文档：

* [Go](https://godoc.org/github.com/prometheus/client\_golang/prometheus#Gauge)
* [Java](https://github.com/prometheus/client\_java#gauge)
* [Python](https://github.com/prometheus/client\_python#gauge)
* [Ruby](https://github.com/prometheus/client\_ruby#gauge)

## Histogram 直方图类型 <a href="#histogram" id="histogram"></a>

_Histogram_ 对观测值(通常是请求持续时间或响应大小之类的数据)进行采样，并将其计数在可配置的数值区间中。它也提供了所有数据的总和。

基本数据指标名称为`<basename>`的直方图类型数据指标，在数据采集期间会显示多个时间序列：

* 数值区间的累计计数器，显示为`<basename>_bucket{le="<数值区间的上边界>"}`
* 所有观测值的总和，显示为`<basename>_sum`
* 统计到的事件计数，显示为`<basename>_count`(与上述`<basename>_bucket{le="+Inf"}`相同)

使用[`histogram_quantile()`函数](https://github.com/hulining/prometheus\_docs/blob/release-v2.19/concepts/functions.md#histogram\_quantile)可以根据直方图及聚合直方图来计算分位数。直方图也适用于计算 [Apdex 得分](https://en.wikipedia.org/wiki/Apdex)。在数值区间操作时，请注意直方图是[累积的](https://en.wikipedia.org/wiki/Histogram#Cumulative\_histogram)。更多直方图用法的详细信息及与 summary 的差异，请参见[直方图和 summary](https://github.com/hulining/prometheus\_docs/blob/release-v2.19/concepts/histograms.md)。

Histogram 的客户端库使用文档：

* [Go](https://godoc.org/github.com/prometheus/client\_golang/prometheus#Histogram)
* [Java](https://github.com/prometheus/client\_java#histogram)
* [Python](https://github.com/prometheus/client\_python#histogram)
* [Ruby](https://github.com/prometheus/client\_ruby#histogram)

## Summary 汇总类型 <a href="#summary" id="summary"></a>

类似于 _histogram_，_summary_ 会采样观察结果(通常是请求持续时间和响应大小之类的数据)。它不仅提供了观测值的总数和所有观测值的总和，还可以计算滑动时间窗口内的可配置分位数。

基本数据指标名称为`<basename>`的 summary 类型数据指标，在数据采集期间会显示多个时间序列：

* 流观察到的事件的 **φ-quantiles**(0≤φ≤1)，显示为`<basename>{quantile="<φ>"}`
* 所有观测值的**总和**，显示为`<basename>_sum`
* 观察到的事件**计数**，显示为`<basename>_count`

有关 φ-quantiles 的详细说明，summary 使用方法用法以及与的差异，请参见 [histograms and summaries](../practices/histograms.md)。

Summary 的客户端库使用文档：

* [Go](https://godoc.org/github.com/prometheus/client\_golang/prometheus#Summary)
* [Java](https://github.com/prometheus/client\_java#summary)
* [Python](https://github.com/prometheus/client\_python#summary)
* [Ruby](https://github.com/prometheus/client\_ruby#summary)
