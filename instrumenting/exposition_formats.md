---
title: 公开格式
sort_rank: 6
---

# 公开格式

可以使用[基于文本](exposition_formats.md#text-based-format)的简单展示格式将数据指标暴露给 Prometheus。有多种[客户端库](https://prometheus.io/docs/instrumenting/clientlibs/)可以为您实现这种格式。如果您的首选语言没有客户端库，则可以[创建自己的客户端库](writing_clientlibs.md)。

{% hint style="info" %}
NOTE: Prometheus的某些早期版本除了支持当前基于文本的格式外，还支持基于[Protocol Buffers](https://developers.google.com/protocol-buffers/)\(又称 Protobuf\)的展示格式。但是，从2.0版开始，Prometheus 不再支持基于 Protobuf 的格式。您可以在[本文档](https://github.com/OpenObservability/OpenMetrics/blob/master/markdown/protobuf_vs_text.md)中了解此更改背后的原因
{% endhint %}

## 基于文本的格式 <a id="text-based-format"></a>

从 Prometheus 2.0 版开始，所有向 Prometheus 公开指标的程序都需要使用基于文本的格式。在本节中，您可以找到有关此格式的一些[基本信息](https://prometheus.io/docs/instrumenting/exposition_formats/#basic-info)以及该格式的[更详细的细节](https://prometheus.io/docs/instrumenting/exposition_formats/#text-format-details)。

### 基本信息 <a id="basic-info"></a>

<table>
  <thead>
    <tr>
      <th style="text-align:left">Aspect</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Inception</b>
      </td>
      <td style="text-align:left">April 2014</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Supported in</b>
      </td>
      <td style="text-align:left">Prometheus version <code>&gt;=0.4.0</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Transmission</b>
      </td>
      <td style="text-align:left">HTTP</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Encoding</b>
      </td>
      <td style="text-align:left">UTF-8, <code>\n</code> line endings</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>HTTP <code>Content-Type</code></b>
      </td>
      <td style="text-align:left"><code>text/plain; version=0.0.4</code> (A missing <code>version</code> value
        will lead to a fall-back to the most recent text format version.)</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Optional HTTP <code>Content-Encoding</code></b>
      </td>
      <td style="text-align:left"><code>gzip</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Advantages</b>
      </td>
      <td style="text-align:left">
        <ul>
          <li>Human-readable</li>
          <li>Easy to assemble, especially for minimalistic cases (no nesting required)</li>
          <li>Readable line by line (with the exception of type hints and docstrings)</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Limitations</b>
      </td>
      <td style="text-align:left">
        <ul>
          <li>Verbose</li>
          <li>Types and docstrings not integral part of the syntax, meaning little-to-nonexistent
            metric contract validation</li>
          <li>Parsing cost</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Supported metric primitives</b>
      </td>
      <td style="text-align:left">
        <ul>
          <li>Counter</li>
          <li>Gauge</li>
          <li>Histogram</li>
          <li>Summary</li>
          <li>Untyped</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>### 文本格式详情 <a id="text-format-details"></a>

Prometheus 基于文本的格式是面向行的。行由换行符\(`\n`\)分隔。最后一行必须以换行符结尾。空行将被忽略。

#### 行的格式 <a id="line-format"></a>

在一行中，标记可以用任意数量的空格和/或制表符分隔\(如果不与先前的标记合并，则必须至少用一个空格分隔\)。行首和行尾的空格将被忽略。

#### 注释，帮助和类型信息 <a id="comments-help-text-and-type-information"></a>

以`#`作为第一个非空白字符的行是注释。除非`#`之后的第一个标记为`HELP`或`TYPE`，否则将忽略它们。这些行的处理方式如下：如果标记为`HELP`，则至少应再有一个是数据指标名称的标记。所有其余标记都被视为该数据指标名称的文档字符串。`HELP`行可以包含任何的 UTF-8 字符\(在数据指标名称之后\)，但是反斜杠和换行符必须分别转义为`\\`和`\n`。对于任何给定的度量标准名称，只能存在一个`HELP`行。

如果标记是`TYPE`，则应该刚好再有两个标记。第一个是数据指标名称，第二个是`counter`, `gauge`, `histogram`, `summary`或`untyped`，用于定义该数据指标的类型。对于给定的数据指标名称，只能存在一个`TYPE`行。 数据指标名称的`TYPE`行必须出现在的第一个报告的数据指标样本之前。如果数据指标名称没有`TYPE`行，则将类型设置为`untyped`。

其余各行使用以下语法\([EBNF](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_form)\)描述样本\(每行一个\):

```text
metric_name [
  "{" label_name "=" `"` label_value `"` { "," label_name "=" `"` label_value `"` } [ "," ] "}"
] value [ timestamp ]
```

在示例语法中:

* `metric_name`和`label_name`遵循常见的 Prometheus 表达式语言限制
* `label_value`可以是任意 UTF-8 字符, 但是反斜线\(`\`\), 双引号\(`"`\)和换行符\(`\n`\)必须分别被转义为`\\`, `\"`和`\n`。
* `value`是按 Go 的[`ParseFloat()`](https://golang.org/pkg/strconv/#ParseFloat)函数要求的浮点数。除标准数值外，`Nan`,`+Inf`和`-Inf`是有效值，分别代表不是数字，正无穷大和负无穷大。
* `timestamp`是一个`int64`数字\(自计时开始的毫秒数，如 1970-01-01 00:00:00 UTC,不包括闰秒\)，符合 Go 的[`ParseInt()`](https://golang.org/pkg/strconv/#ParseInt)函数要求。

#### 分组和排序 <a id="grouping-and-sorting"></a>

给定指标的所有行都必须作为一个独立的组提供，并首先提供可选的`HELP`和`TYPE`行\(无特定顺序\)。除此之外，在重复展示中重新排序分类是优选的，但不是必需的，即，如果计算成本过高，则不要排序。

每行必须有数据指标名称和标签的唯一组合。否则，采集行为是不确定的。

#### Histograms and summaries

`histogram`和`summary`类型的数据指标很难用文本格式表示。以下是通用的约定:

* 名为`x`的`histogram`或`summary`类型的数据指标总和以名为`x_sum`的独立样本给出。
* 名为`x`的`histogram`或`summary`类型的数据指标计数以名为`x_count`的独立样本给出。
* 名为`x`的`summary`类型的数据指标的每个分位数\(quantile\)以相同名称`x`及带有标签`{quantile="y"}`的独立样本给出。
* 名为`x`的`histogram`类型的数据指标的每个存储桶计数以名为`x_bucket`及带有标签`{le="y"}`的独立样本给出\(`y`是每个存储桶的上限\)。
* `histogram`类型的数据指标_必须_包含一个带有`{le="+Inf"}`标签的独立样本。其值必须与`x_count`的值相同
* 名为`x`的`histogram`或`summary`类型的数据指标总和作为名为`x_sum`的单独样本给出。
* 的数据指标的存储桶\(`histogram`类型\)或分位数\(`summary`类型\)必须按照其标签值\(分别用于`le`或`quantile`标签\)的递增顺序出现。

### 文本格式的示例 <a id="text-format-example"></a>

以下是一个完整的 Prometheus 数据指标展示的示例，包括注释，`HELP`和`TYPE`表达式，histogram、summary 类型的数据、字符转义等示例

```text
# HELP http_requests_total The total number of HTTP requests.
# TYPE http_requests_total counter
http_requests_total{method="post",code="200"} 1027 1395066363000
http_requests_total{method="post",code="400"}    3 1395066363000

# Escaping in label values:
msdos_file_access_time_seconds{path="C:\\DIR\\FILE.TXT",error="Cannot find file:\n\"FILE.TXT\""} 1.458255915e9

# Minimalistic line:
metric_without_timestamp_and_labels 12.47

# A weird metric from before the epoch:
something_weird{problem="division by zero"} +Inf -3982045

# A histogram, which has a pretty complex representation in the text format:
# HELP http_request_duration_seconds A histogram of the request duration.
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.05"} 24054
http_request_duration_seconds_bucket{le="0.1"} 33444
http_request_duration_seconds_bucket{le="0.2"} 100392
http_request_duration_seconds_bucket{le="0.5"} 129389
http_request_duration_seconds_bucket{le="1"} 133988
http_request_duration_seconds_bucket{le="+Inf"} 144320
http_request_duration_seconds_sum 53423
http_request_duration_seconds_count 144320

# Finally a summary, which has a complex representation, too:
# HELP rpc_duration_seconds A summary of the RPC duration in seconds.
# TYPE rpc_duration_seconds summary
rpc_duration_seconds{quantile="0.01"} 3102
rpc_duration_seconds{quantile="0.05"} 3272
rpc_duration_seconds{quantile="0.5"} 4773
rpc_duration_seconds{quantile="0.9"} 9001
rpc_duration_seconds{quantile="0.99"} 76656
rpc_duration_seconds_sum 1.7560473e+07
rpc_duration_seconds_count 2693
```

## 历史版本 <a id="historical-versions"></a>

有关历史格式版本的详细信息，请参阅旧版[Client Data Exposition Format](https://docs.google.com/document/d/1ZjyKiKxZV83VI9ZKAXRGKaUKK2BIWCT7oiGBKDBpjEY/edit?usp=sharing)文档。

