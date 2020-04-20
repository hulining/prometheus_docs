---
title: PROMETHEUS 查询
---

# Prometheus  查询

Prometheus 提供了一种名为 PromQL\(Prometheus Query Language\) 的功能查询语言，使用户可以实时选择和汇总时间序列数据。，表达式的结果可以在 Prometheus 的表达式浏览器中显示为图形和表格数据，也可以由外部系统通过 [HTTP API](api.md) 使用

## 示例 <a id="examples"></a>

本文档仅供参考。对于学习，从几个[示例](examples.md)开始可能会更容易。

## 表达式语言数据类型 <a id="expression-language-data-types"></a>

在 Prometheus 表达式的表达语言中，一个表达式或子表达式可以计算为以下四种类型之一：

* **instant vector\(瞬时/即时向量\)**：一组时间序列，每个时间序列包含一个样本，所有数据样本共享相同的时间戳。
* **Range vector\(范围向量\)**：一组时间序列，其中包含每个时间序列随时间变化的一系列数据点
* **Scalar\(标量\)**：一个简单的数字浮点值
* **String\(字符串\)**：一个简单的字符串值。目前未使用

根据使用场景\(如，绘制图形或显示表达式的输出时\)，用户执行的表达式的结果，只有某些类型是合法的。例如，返回即时向量的表达式是唯一可以直接绘制图形的类型。

## 字面量 <a id="literals"></a>

### 字符串字面量 <a id="string-literals"></a>

字符串可以由单引号，双引号或反引号指定为字符串字面量

PromQL 与 Go 遵循相同的[转义规则](https://golang.org/ref/spec#String_literals)。在单引号或双引号中，以反斜杠开始一个转义序列，其后可以跟`a`, `b`, `f`, `n`, `r`, `t`, `v`或 `\`。可以使用八进制\(`\nnn`\)或十六进制\(`\xnn`, `\unnn`和`\Unnnnnnnn`\)提供特定字符。

在反引号内不处理转义字符，与 Go 不同，Prometheus 不会丢弃反引号中的换行符。如：

```go
"this is a string"
'these are unescaped: \n \\ \t'
`these are not unescaped: \n ' " \t`
```

### 浮点型字面量 <a id="float-literals"></a>

标量浮点值可以表示为`[-](digits)[.(digits)]`形式

```text
- 2.43
```

## 时间序列选择器 <a id="time-series-selectors"></a>

### 即时向量选择器 <a id="instant-vector-selectors"></a>

瞬时向量选择器允许在给定时间戳\(瞬时\)上选择一组时间序列和每个样本的当个采样值：在最简单的形式中，仅指定度量名称。这会生成包含具有该数据指标名称的所有时间序列的元素的即时向量。

下面这个例子选择所有数据指标名称为`http_requests_total`的时间序列样本数据：

```text
http_requests_total
```

通过在花括号\(`{}`\)中添加以逗号分割的标签匹配器列表，可以进一步过滤这些时间序列。

此示例仅选择将`job`标签设置为`prometheus`，`group`标签设置为`canary`且数据指标名称为`http_requests_total`的时间序列样本数据：

```text
http_requests_total{job="prometheus",group="canary"}
```

也可以反向匹配标签值，或将标签值与正则表达式匹配。标签匹配操作符如下所示：

* `=`: 选择与提供的字符串完全相同的标签\(精确匹配\)
* `!=`: 选择不等于提供的字符串的标签\(反向匹配\)
* `=~`: 选择与提供的字符串进行正则表达式匹配的标签\(正则表达式匹配\)
* `!~`: 选择正则表达式不匹配提供的字符串的标签\(反向正则表达式匹配\)

例如，选择所有`staging`, `testing`, `development`环境且 HTTP 请求方法不是`GET`的`http_requests_total`时间序列

```text
http_requests_total{environment=~"staging|testing|development",method!="GET"}
```

空值匹配标签匹配器还会选择所偶根本没有设置指定标签的时间序列。正则表达式匹配完全锚定，同一标签名称可能有多个匹配器。

向量选择器必须指定一个名称或至少一个与空字符串不匹配的标签匹配器。 以下表达式是非法的：

```text
{job=~".*"} # Bad!
```

相比之下，这些表述是合法的，因为他们都有一个选择不匹配空标签值。

```text
{job=~".+"}              # Good!
{job=~".*",method="get"} # Good!
```

标签匹配器可以被应用到数据指标名称，使用`__name__`标签筛选数据指标名称。例如：表达式`http_requests_total`等价于`{__name__="http_requests_total"}`。匹配器也可以是使用除`=`之外的匹配规则\(如`!=`, `=~`, `!~`\)。下面的表达式筛选了数据指标名称以`job:`开头的时间序列数据：

```text
{__name__=~"job:.*"}
```

数据指标名称不可以是`bool`, `on`, `ignoring`, `group_left`和`group_right`关键字。如下表达式是非法的：

```text
on{} # Bad!
```

针对此限制的解决方法是使用`__name__`标签：

```text
{__name__="on"} # Good!
```

Prometheus中的所有正则表达式都使用[RE2语法](https://github.com/google/re2/wiki/Syntax)。

### 范围向量选择器 <a id="range-vector-selectors"></a>

范围向量的工作方式与即时向量基本相同，不同之处在于，范围向量从当前瞬间选择了一定范围的样本。语法上，将范围持续时间附加在向量选择器末尾的方括号\(`[]`\)中，以指定为每个范围向量元素提取多久的时间值。

持续时间指定为数字，紧随其后的是以下单位之一：

* `s` - 秒
* `m` - 分钟
* `h` - 小时
* `d` - 天
* `w` - 周
* `y` - 年

在此示例中，我们选择在过去 5 分钟，数据指标名称为`http_requests_total`且`job`标签为`prometheus`的所有时间序列记录的所有值：

```text
http_requests_total{job="prometheus"}[5m]
```

### 偏移量 <a id="offset-modifier"></a>

`offset`修饰符允许更改查询中各个瞬时向量和范围向量的时间偏移。

例如，以下表达式返回相对于当前查询时间过去 5 分钟的 `http_requests_total` 的值：

```text
http_requests_total offset 5m
```

请注意，`offset`修饰符始必须直接跟在选择器后面，以下内容是正确的：

```text
sum(http_requests_total{method="GET"} offset 5m) // GOOD.
```

下面这种情况是不正确的:

```text
sum(http_requests_total{method="GET"}) offset 5m // INVALID.
```

范围向量同样适用。下面示例返回`http_requests_total`一周前的 5 分钟速率：

```text
rate(http_requests_total[5m] offset 1w)
```

## 子查询 <a id="subquery"></a>

子查询允许您针对给定的范围和分辨率运行即时查询。子查询的结果是范围向量。

语法：`<instant_query> '[' <range> ':' [<resolution>] ']' [ offset <duration> ]`

* `<resolution>` 是可选的。默认是全局评估时间间隔。

## 操作符 <a id="operators"></a>

Prometheus 支持二元和聚合操作符。详见[表达式语言操作符](operators.md)

## 函数 <a id="functions"></a>

Prometheus 提供了一些函数列表操作时间序列数据。详见[表达式语言函数](functions.md)

## 注释 <a id="comments"></a>

PromQL 支持以`#`开头的行注释。 例：

```text
# This is a comment
```

## 注意事项 <a id="gotchas"></a>

### Staleness

运行查询时，将使用采样数据的时间戳，而不依赖当前实际时间序列数据。这主要是为了支持诸如聚合\(`sum`，`avg`等\)过程中，多个聚合时间序列在时间上不完全一致情况。由于它们的独立性，Prometheus 需要在这些时间戳上为每个相关时间序列分配一个值。只需在此时间戳之前获取最新样本即可。

如果目标数据采集或规则评估不再返回先前存在的时间序列的样本，则该时间序列将被标记为 stale。如果删除了目标，则其先前返回的时间序列将在不久后标记为 stale。

如果在某个时间序列标记为 stale 后以采样时间戳评估查询，则该时间序列不会返回任何值。如果随后在该时间序列中摄取了新样本，则它们将照常返回。

如果在采样时间戳记之前 5 分钟\(默认情况下\)未找到任何采样，则在该时间点该时间序列将不返回任何值。这实际上意味着，在最新采集的样本早于 5 分钟或标记为 stale 之后，时间序列将从图表中"消失"。

对于在其采集中包含时间戳的时间序列，不会标记陈旧性。在这种情况下，仅应用 5 分钟的阈值。

### 避免慢查询和过载 <a id="avoiding-slow-queries-and-overloads"></a>

如果查询需要处理大量数据，对其进行图形化处理可能会超时或使服务器或浏览器超载。因此，在对未知数据构建查询时，请先在 Prometheus 表达式浏览器的表格视图中构建查询，直到结果集看起来合理为止\(最多数百个时间序列，而不是数千个\)。当您充分过滤或汇总了数据后，才切换到图形模式。如果该表达式仍花费很长时间来绘制图形，请通过[记录规则](../configuration/recording_rules.md#recording-rules)将其预先记录。

这对于 Prometheus 查询语言尤其重要。在该语言中，像`api_http_requests_total`这样的裸指标名称选择器可以扩展到成千上万个具有不同标签的时间序列。还请记住，即使输出只是少量时间序列，在多个时间序列上聚合的表达式也会在服务器上产生负载。这类似于将关系数据库中列的所有值相加会很慢，即使输出值只是一个数字也是如此。

