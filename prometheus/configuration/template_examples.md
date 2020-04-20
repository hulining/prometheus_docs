---
title: 模版示例
---

# 模板示例

Prometheus支持在告警的注解和标签以及服务的控制台页面中进行模板化。模板具有对本地数据库运行查询，数据迭代，条件判断，格式化数据等功能。Prometheus模板语言基于 [Go 模板](https://golang.org/pkg/text/template/)系统。

## 简单告警字段模版 <a id="simple-alert-field-templates"></a>

```yaml
alert: InstanceDown
expr: up == 0
for: 5m
labels:
  severity: page
annotations:
  summary: "Instance {{$labels.instance}} down"
  description: "{{$labels.instance}} of job {{$labels.job}} has been down for more than 5 minutes."
```

警报字段模板在触发告警期间的每个规则迭代执行，因此请保持所有查询和模板的轻量级。如果您需要更复杂的告警模板，建议改为链接到控制台。

## 简单迭代 <a id="simple-iteration"></a>

以下将显示实例列表及它们是否启动：

```yaml
{{ range query "up" }}
  {{ .Labels.instance }} {{ .Value }}
{{ end }}
```

特殊的`.`变量包含每次循环迭代的当前样本值

## 显示一个值 <a id="display-one-value"></a>

```yaml
{{ with query "some_metric{instance='someinstance'}" }}
  {{ . | first | value | humanize }}
{{ end }}
```

Go 和 Go 的模板语言都是强类型的，因此必须检查返回的样本以避免执行错误。如，如果采集或规则尚未运行，或者主机已关闭，则可能发生这种情况。

随附的`prom_query_drilldown`模板可以处理此类问题，允许格式化结果并链接到[表达式浏览器](../../visualization/browser.md)。

## 使用控制台 URL 参数 <a id="using-console-url-parameters"></a>

```yaml
{{ with printf "node_memory_MemTotal{job='node',instance='%s'}" .Params.instance | query }}
  {{ . | first | value | humanize1024 }}B
{{ end }}
```

如果以`console.html?instance=hostname`进行访问，`.Params.instance`将被赋值为`hostname`。

## 高级迭代 <a id="advanced-iteration"></a>

```yaml
<table>
{{ range printf "node_network_receive_bytes{job='node',instance='%s',device!='lo'}" .Params.instance | query | sortByLabel "device"}}
  <tr><th colspan=2>{{ .Labels.device }}</th></tr>
  <tr>
    <td>Received</td>
    <td>{{ with printf "rate(node_network_receive_bytes{job='node',instance='%s',device='%s'}[5m])" .Labels.instance .Labels.device | query }}{{ . | first | value | humanize }}B/s{{end}}</td>
  </tr>
  <tr>
    <td>Transmitted</td>
    <td>{{ with printf "rate(node_network_transmit_bytes{job='node',instance='%s',device='%s'}[5m])" .Labels.instance .Labels.device | query }}{{ . | first | value | humanize }}B/s{{end}}</td>
  </tr>{{ end }}
</table>
```

在这个示例中，我们遍历所有网络设备并显示每个网络设备的网络流量。

由于`range`操作未指定变量，因此`.Params.instance`在循环内不可用，`.`是循环变量。

## 定义可重用的模版 <a id="defining-reusable-templates"></a>

Prometheus 支持定义可重复使用的模板。与[控制台库](template_reference.md#console-templates)支持结合使用时，此功能特别强大，允许在各个控制台之间共享模板。

```yaml
{{/* Define the template */}}
{{define "myTemplate"}}
  do something
{{end}}

{{/* Use the template */}}
{{template "myTemplate"}}
```

模板被限制于仅支持一个参数。 `args`函数可用于包装多个参数。

```yaml
{{define "myMultiArgTemplate"}}
  First argument: {{.arg0}}
  Second argument: {{.arg1}}
{{end}}
{{template "myMultiArgTemplate" (args 1 2)}}
```

