---
title: 控制台模板
---

# 控制台模板

控制台模板允许使用 [Go 模板语言](https://golang.org/pkg/text/template/)创建任意控制台。这些是由 Prometheus 服务提供的。

控制台模板是创建可在源代码管理中轻松管理模板的最强大方法。不过这是一条学习曲线，因此刚接触这种监控方式的用户应首先尝试使用 [Grafana](grafana.md)。

## 快速开始 <a id="getting-started"></a>

Prometheus 附带了一个控制台示例，可以给您一些帮助。可以在正在运行的 Prometheus 的`/consoles/index.html.example`上找到这些文件，如果 Prometheus 正在采集带有`job="node"`标签的 Node Exporter，则会显示 Node Exporter 控制台。

示例控制台包含 5 部分： 1. 顶部的导航栏 2. 左边的菜单栏 3. 底部的时间控制栏 4. 中间的主要内容，通常是图形 5. 右侧的表

导航栏用于链接到其它系统，例如其它 [Prometheis](../introduction/faq.md#what-is-the-plural-of-prometheus)，文档以及对您有意义的任何其它内容。菜单用于在相同 Prometheus 服务中进行导航，这对于能够快速打开一个关联其它信息选项卡的控制台非常有用。两者都在`console_libraries/menu.lib`中配置。

时间控件允许更改图表的窗口时间和范围。控制台 URL 可以共享且将显示相同的图形。

通常主要内容是图形。提供了一个可配置的 JavaScript 图形库，该库处理来自 Prometheus 的请求数据，并通过[Rickshaw](https://shutterstock.github.io/rickshaw/)进行渲染后展示出来。

最后，右侧的表格可以用来呈现比图形更紧凑的统计信息。

## 控制台示例 <a id="example-console"></a>

以下是一个基本的控制台，它在右侧表格中显示了任务数，正常运行数，CPU 平均使用率和内存平均使用率。主要内容包含一个每秒查询的图。

```text
{{template "head" .}}

{{template "prom_right_table_head"}}
<tr>
  <th>MyJob</th>
  <th>{{ template "prom_query_drilldown" (args "sum(up{job='myjob'})") }}
      / {{ template "prom_query_drilldown" (args "count(up{job='myjob'})") }}
  </th>
</tr>
<tr>
  <td>CPU</td>
  <td>{{ template "prom_query_drilldown" (args
      "avg by(job)(rate(process_cpu_seconds_total{job='myjob'}[5m]))"
      "s/s" "humanizeNoSmallPrefix") }}
  </td>
</tr>
<tr>
  <td>Memory</td>
  <td>{{ template "prom_query_drilldown" (args
       "avg by(job)(process_resident_memory_bytes{job='myjob'})"
       "B" "humanize1024") }}
  </td>
</tr>
{{template "prom_right_table_tail"}}


{{template "prom_content_head" .}}
<h1>MyJob</h1>

<h3>Queries</h3>
<div id="queryGraph"></div>
<script>
new PromConsole.Graph({
  node: document.querySelector("#queryGraph"),
  expr: "sum(rate(http_query_count{job='myjob'}[5m]))",
  name: "Queries",
  yAxisFormatter: PromConsole.NumberFormatter.humanizeNoSmallPrefix,
  yHoverFormatter: PromConsole.NumberFormatter.humanizeNoSmallPrefix,
  yUnits: "/s",
  yTitle: "Queries"
})
</script>

{{template "prom_content_tail" .}}

{{template "tail"}}
```

`prom_right_table_head`和`prom_right_table_tail`模板包裹右侧的表。这是可选的

`prom_query_drilldown`是一个模板，它将计算传递给它的表达式，对其进行格式设置病链接到[表达式浏览器](browser.md)中的表达式。第一个参数是表达式，第二个参数是要使用的单位，第三个参数是如何格式化输出。仅第一个参数是必需的。

对于`prom_query_drilldown`第三个参数合法的输出格式：

* 没有指定: 默认 Go 语言格式的输出
* `humanize`: 使用[metric prefixes](https://en.wikipedia.org/wiki/Metric_prefix)显示结果
* `humanizeNoSmallPrefix`: 对于绝对值大于 1 的值，使用[metric prefixes](https://en.wikipedia.org/wiki/Metric_prefix)显示结果；对于绝对值小于 1 的值，显示 3 位有效数字。这对于避免在`humanize`过程中产生千分制的单位很有用。
* `humanize1024`: 使用 1024 而不是 1000 作为底数显示可读性转化结果。这通常用作将`B`做为第二个参数使用，以产生可读性更好的单位如，`KiB`，`Mi`B
* `printf.3g`: 显示 3 位有效数字

还可以定义自定义格式。有关示例，请参见 [prom.lib](https://github.com/prometheus/prometheus/blob/master/console_libraries/prom.lib)。

## 图形库 <a id="graph-library"></a>

图形库的调用方式为:

```text
<div id="queryGraph"></div>
<script>
new PromConsole.Graph({
  node: document.querySelector("#queryGraph"),
  expr: "sum(rate(http_query_count{job='myjob'}[5m]))"
})
</script>
```

`head`模板加载需要的 Javascript 和 CSS.

图形库的参数如下:

| 名称 | 解释 |
| :--- | :--- |
| expr | 必需。图形的表达式。可以是一个列表 |
| node | 必需。要渲染到的 DOM 节点。 |
| duration | 可选。图形的持续时间\(窗口时间\)。默认 1 小时 |
| endTime | 可选。图形的结果 Unix 时间。默认为当前时间 |
| width | 可选。图形的宽度，不包含标题。默认自动适应 |
| height | 可选。图形的高度，不包含标题和图例。默认 200 像素 |
| min | 可选。x 轴最小的值。默认是数据的最小值 |
| max | 可选。y 轴的最大值。默认是数据的最大值 |
| renderer | 可选。图形的类型。可选值为`line`和`area`\(堆叠图\)。默认是`line` |
| name | 可选。图例标题中图例和悬停的详细信息。如果传入字符串，`[[label]]`将替换为标签值。如果传入函数，它将传递一个标签映射，并将名称作为字符串返回。可以是列表 |
| xTitle | 可选。x 轴标题。默认是`Time` |
| yUnits | 可选。y 轴单位。默认为空 |
| yTitle | 可选。y 轴标题。默认为空 |
| yAxisFormatter | 可选。y 轴数字格式。默认为`PromConsole.NumberFormatter.humanize` |
| yHoverFormatter | 可选。悬停详细信息的数字格式。默认为`PromConsole.NumberFormatter.humanizeExact`。 |
| colorScheme | 可选的。绘图要使用的配色方案。可以是十六进制颜色代码的列表，也可以是 Rickshaw 支持的[颜色方案](https://github.com/shutterstock/rickshaw/blob/master/src/js/Rickshaw.Fixtures.Color.js)。默认为`'colorwheel'` |

如果`expr`和`name`均为列表，它们的长度必须相同，名称将应用于对应表达式的图。

`yAxisFormatter`和`yHoverFormatter`的有效选项:

* `PromConsole.NumberFormatter.humanize`: 使用 [metric prefixes](https://en.wikipedia.org/wiki/Metric_prefix) 进行格式化
* `PromConsole.NumberFormatter.humanizeNoSmallPrefix`: 对于绝对值大于 1 的值，使用[metric prefixes](https://en.wikipedia.org/wiki/Metric_prefix)显示结果；对于绝对值小于 1 的值，显示 3 位有效数字。这对于避免在`PromConsole.NumberFormatter.humanize`过程中产生千分制的单位很有用
* `PromConsole.NumberFormatter.humanize1024`: 使用 1024 而不是 1000 作为底数显示可读性转化结果

