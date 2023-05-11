# 指标和标签命名

使用 Prometheus 不需要本文档中介绍的度量标准和标签约定，但是可以用作风格指南和最佳实践的集合。各个组织可能希望采用其中的一些做法，如命名约定，有所不同。

## 指标名称 <a href="#metric-names" id="metric-names"></a>

指标名称...

* ...必须符合[数据模型](../concepts/data\_model.md#metric-names-and-labels)中的有效字符
* ...应具有与度量标准所属域相关的(单个单词)应用程序前缀。客户端库有时将该前缀称为`namespace`。对于特定于应用程序的数据指标，前缀通常是应用程序名称本身。但是，有时数据指标更为通用，例如客户端库导出的标准化数据指标。例如:
  * **`prometheus`**`_notifications_total`(特指 Prometheus 服务)
  * **`process`**`_cpu_seconds_total`(由许多客户端库导出)
  * **`http`**`_request_duration_seconds`(对于所有的 HTTP 请求)
* ...必须由一个单位(如，不要将秒与毫秒或秒与字节混合)
* ...应当使用基本单位(如. seconds, bytes, meters - 而不是 milliseconds, megabytes, kilometers).参见以下基本单位列表.
* ...应以单位的复数形式进行描述。请注意，如果适用，除单位外，累加计数还具有`total`作为后缀
  * `http_request_duration_`**`seconds`**
  * `node_memory_usage_`**`bytes`**
  * `http_requests_`**`total`**(用于无单位的累计计数)
  * `process_cpu_`**`seconds_total`**(用于带单位的累加计数)
  *   `foobar_build`**`_info`**

      (用于提供有关正在运行的二进制文件的[元数据](https://www.robustperception.io/exposing-the-software-version-to-prometheus)的伪度量)
* ...应该在所有标签维度上表示相同的逻辑事物
  * 请求时长
  * 数据传输字节
  * 瞬时资源使用率(百分比)

根据经验，给定指标所有维度上的`sum()`或`avg()`都应该有意义(尽管不一定有用)。如果没有意义，请将数据分成多个指标。 如，在一个数据指标中具有各种队列的容量是好的，而将队列的容量与队列中当前元素的数量混合则不行。

## 标签 <a href="#labels" id="labels"></a>

使用标签来区分被测物的特征:

* `api_http_requests_total` - 区分请求类型: `operation="create|update|delete"`
* `api_request_duration_seconds` - 区分请求阶段: `stage="extract|transform|load"`

不要将标签名称放在度量标准名称中，这会引入冗余，且如果将各个标签汇总在一起会引起混淆。

警告: 请记住，键值标签对的每个唯一组合都代表一个新的时间序列，这可以大大增加存储的数据量。请勿使用标签存储具有高基数(许多不同的标签值)的维度，例如用户 ID，电子邮件地址或其他无限制的值集。

{% hint style="warning" %}
警告: 请记住，键值标签对的每个唯一组合都代表一个新的时间序列，这可以大大增加存储的数据量。请勿使用标签存储具有高基数(许多不同的标签值)的维度，例如用户 ID，电子邮件地址或其他无限制的值集。
{% endhint %}

## 基本单位 <a href="#base-units" id="base-units"></a>

Prometheus没有任何单位的硬编码。为了获得更好的兼容性，应使用基本单位。以下列出了一些数据指标序列及其基本单位。该列表并不详尽。

|        Family        |   Base unit   |                                    Remark                                   |
| :------------------: | :-----------: | :-------------------------------------------------------------------------: |
|       Time(时间)       |   seconds(秒)  |                                                                             |
|    Temperature(温度)   | celsius(摄氏温度) |                         由于实践原因，_celsius_ 优于 _kelvin_                        |
|      Length(长度)      |   meters(米)   |                                                                             |
|      Bytes(字节数)      |   bytes(字节)   |                                                                             |
|        Bits(位)       |   bytes(字节)   |                 为了避免混淆不同的度量标准，请始终使用 _bytes_，即使 _bits_ 看起来更常见                |
|     Percent(百分比)     |   ratio(比率)   | 值是 0-1(而不是0-100). `ratio` 仅用作 `disk_usage_ratio` 之类的后缀。常用指标名称遵循模式 `A_per_B` |
|      Voltage(电压)     |   volts(伏特)   |                                                                             |
| Electric current(电流) |    amperes    |                                                                             |
|      Energy(能量)      |   joules(焦耳)  |                                                                             |
|       Mass(质量)       |    grams(克)   |                  _grams_ 比 _kilograms_ 更可避免 _kilo_ 前缀出现问题.                  |
