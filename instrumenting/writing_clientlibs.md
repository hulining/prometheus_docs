# 编写客户端库

本文档涵盖了 Prometheus 客户端库应提供的功能和 API，目的是在各个库之间保持一致，从而简化易用案例，并避免提供可能导致用户走错路的功能。

在撰写本文时，已经[支持 10 种语言](clientlibs.md)，我们已经很好地了解了如何编写客户端。这些准则旨在帮助新客户端库的作者创建良好的库。

## 约定 <a href="#conventions" id="conventions"></a>

MUST/MUST NOT/SHOULD/SHOULD NOT/MAY 等关键字的含义已经在 [https://www.ietf.org/rfc/rfc2119.txt](https://www.ietf.org/rfc/rfc2119.txt) 给出

另外，ENCOURAGED 表示某个功能对客户端库来说是希望有的，但是没有也没关系。换句话说，如果有是非常好的。

注意事项:

* 充分发挥每种语言的优点
* 通用样例是很简单
* 做某事的正确方法应该是简单的方法
* 用复杂的用例也是可以的

常见用例如下(按顺序)：

* 没有标签的计数器在库/应用中广泛分布
* Summaries/Histograms 中的时间函数/代码块
* 跟踪事务当前状态(及其极限)的 Gauges
* 批处理作业的监控

## 总体结构 <a href="#overall-structure" id="overall-structure"></a>

客户端必须在内部编写为基于回调的。客户端通常应遵循此处描述的结构。

关键类是`Collector`。它具有返回 0 或多个数据指标及其样本的方法(通常称为`collect`方法)。`Collector`在`CollectorRegistry`中注册。数据以通过将`CollectorRegistry`传递给类/方法/函数`"bridge"`的方式来暴露，并以 Prometheus 支持的格式返回数据指标。每次对`CollectorRegistry`进行采集时，都必须回调到每个`Collector`的`collect`方法。

大多数用户与之交互的接口是 Counter, Gauge, Summary 和 Histogram Collectors。这些代表一个独立的数据指标且应该涵盖用户正在编写代码的绝大多数用例。

更高级的用例(如，另一个监控/集成系统的代理)需要自定义`Collector`。有人可能还想要写一个`"bridge"`，它需要`CollectorRegistry`并以其他监控/集成系统可以理解的格式生成数据，从而使用户只需要考虑一个集成系统即可。

`CollectorRegistry`应当提供`register()/unregister()`函数，`Collector`应当被允许注册到多个`CollectorRegistry`

客户端库必须是线程安全的。

对于非面向对象语言(如 C)，客户端库应尽可能的遵循这种结构的思想。

### 命名 <a href="#naming" id="naming"></a>

客户端库应该遵循本文档中提到的函数/方法/类名称，同时要记住它们使用的语言的命名约定。例如，`set_to_current_time()`适合在 Python 中命名一个方法，而在 Go 中`SetToCurrentTime()`则更好，`setToCurrentTime()`是Java中的约定。如果名称由于技术原因而有所不同(例如，不允许函数重载)，则文档/帮助字符串应将用户指向其他名称。

客户端库不得提供与此处给出的名称相同或相似但具有不同的语义的函数/方法/类。

## 数据指标 <a href="#metrics" id="metrics"></a>

`Counter`, `Gauge`, `Summary`和`Histogram`[数据指标类型](../concepts/metric\_types.md)是面向用户的主要接口。

`Counter`和`Gauge`必须是客户端库的一部分。必须提供`Summary`和`Histogram`其中之一。

这些变量应主要用作文件静态变量，即与要集成的代码在同一文件中定义的全局变量。客户端应该启用它。常见的用例是整体集成一段代码，而不是在一个对象实例的上下文中集成的一段代码。用户不必担心在整个代码中探测其数据指标，客户端应该为其做这些(如果没有，用户将在库周围编写一个包装器以使其更容易 - 而很少趋于顺利)。

必须有一个默认的`CollectorRegistry`，标准数据指标必须默认隐式地注册到其中，无需用户进行任何特殊工作。必须有一个方法可以使数据指标不注册到默认的`CollectorRegistry`中，以供批处理作业和单元测试使用。自定义的`Collector`也应遵循此规则。

确切地如何创建数据指标因语言而异。对于某些而言(Java, Go)，构建器是最好的。而对于另一些(Python)，函数参数足够丰富，可以一次调用。

例如，在 Java `Simpleclient`中，我们有:

```java
class YourClass {
  static final Counter requests = Counter.build()
      .name("requests_total")
      .help("Requests.").register();
}
```

这将使用默认的`CollectorRegistry`注册请求。通过调用`build()`而不是`register()`该数据指标将不会被注册(对于单元测试很方便)，您也可以将`CollectorRegistry`传递给`register()`(对于批处理作业很方便)。

### Counter

[Counter](../concepts/metric\_types.md#counter) 是单调递增的计数器。它必须不允许该值减小，但可以将其重置为 0(例如服务重新启动)。

Counter 必须有如下方法：

* `inc()`: 将计数器加 1
* `inc(double v)`: 计数器增加指定的值。必须检查 `v>=0`

Counter 最好有:

统计在给定代码段中抛出/引发异常的方法，并且可以选择计数某些类型的异常。这类似于 Python 中的`count_exceptions`。

计数器必须从 0 开始。

### Gauge

[Gauge](../concepts/metric\_types.md#gauge) 表示可以升降的值。

Gauge 必须有如下方法:

* `inc()`: 将值增加 1
* `inc(double v)`: 增加指定的值
* `dec()`: 将值减少 1
* `dec(double v)`: 减少指定的值
* `set(double v)`: 设置为指定的值

Gauge 必须从 0 开始，您可以提供从其它数字开始的方法。

Gauge 应该有如下方法:

* `set_to_current_time()`: 将 gauge 设置为当前 Unix 时间(以秒为单位)

Gauge 最好有:

跟踪某些代码/函数中正在进行的请求的方法。这类似于 Python 中的`track_inprogress`。

代码计时并将 gauge 设定为它所持续时间(以秒为单位)的方法。这对于批处理作业很有用。这类似于 Java 中的`startTimer/setDuration`,Python 中的`time()`装饰器/上下文管理器。这应该遵循与 Gauge/Histogram 具有相同的模式(虽然是`set()`而不是`observe()`)。

### Summary

[Summary](../concepts/metric\_types.md#summary) 会在滑动的时间窗口内对观察结果(通常是请求时间之类的)进行采样，并即时了解其分布，频率和总和。

Summary 必须禁止用户将`quantile`设置为标签名称，因为该名称在内部用于指定摘要分位数。Summary 鼓励提供 quantiles 作为暴露数据，尽管这些数据不能汇总，而且往往很慢。 Summary 必须不允许有 quantiles，因为`_count`/`_sum`非常有用，并且必须是默认值。

Summary 必须有如下方法:

* `observe(double v)`: 观察给定的数量。

Summary 应该包含如下方法:

为用户代码计时的方法，以秒为单位。这类似于 Python 中的`time()`装饰器/上下文管理器，Java 中的`startTimer/setDuration`。绝不能提供秒以外的单位(如果用户需要，他们可以手工完成)。 这应该遵循与 Gauge/Histogram 具有相同的模式。

Summary 中的`_count`/`_sum`必须从 0 开始。

### Histogram

[Histogram](../concepts/metric\_types.md#histogram) 允许事件的汇总分布，例如请求延迟。这是每个桶的计数器的核心。

Histogram 禁止将`le`作为用户设置的标签，因为`le`在内部用于指定存储桶。

Histogram 必须提供一种手动选择存储桶的方法。应该提供以`linear(start, width, count)`和`exponential(start, factor, count)`方式设置存储桶的方法。计数必须排除`+Inf`存储桶。

Histogram 应具有与其他客户端库相同的默认存储区。指标一旦创建，便不得更改。

Histogram 必须有如下方法:

* `observe(double v)`: 观察给定的数量。

Histogram 应该包含如下方法:

为用户代码计时的方法，以秒为单位。这类似于 Python 中的`time()`装饰器/上下文管理器，Java 中的`startTimer/setDuration`。绝不能提供秒以外的单位(如果用户需要，他们可以手工完成)。 这应该遵循与 Gauge/Summary 具有相同的模式。

**其他指标的注意事项**

为其他语言提供超出上述说明范围的其他功能是值的鼓励的。

如果存在常见用例，则可以简化它，只要它不会鼓励不良行为(例如，次优指标/标签布局或在客户端进行计算)。

### 标签 <a href="#labels" id="labels"></a>

标签是 Prometheus 最强大的功能之一，但很容易被滥用。因此，客户端库在如何向用户提供标签时必须非常小心。

在任何情况下，客户端库都绝对不允许用户为`Gauge/Counter/Summary/Histogram`的同一数据指标或客户端库提供的任何`Collector`使用不同的标签名称。

来自自定义 Collector 的数据指标集合应该始终具有一致的标签名称。由于仍然有少数情况但有效的用例并非如此，因此客户端库不应对此进行验证。

标签虽然功能强大，但大多数数据指标都没有标签。因此，API 应该允许标签但是不能强制它。

客户端库必须允许在创建 Gauge/Counter/Summary/Histogram 时可选地指定标签名称的列表。客户端库应该支持任意数量的标签名称。 客户端库必须验证标签名称是否满足[文档要求](../concepts/data\_model.md#metric-names-and-labels)。

提供对数据指标的标签维度的访问的一般方法是通过`labels()`方法，该方法获取标签值的列表或从标签名称到标签值的映射，并返回"Child"。然后可在 Child 上调用一些如`.inc()`/`.dec()`/`.observe()`常规方法。

`labels()`方法返回的 Child 应该是用户可缓存的，以避免不得不再次查找 - 这在关键延迟的代码中很重要。

带有标签的数据指标应支持具有与`labels()`相同签名的`remove()`方法，该方法将不从数据指标中删除子集，而`clear()`方法将从该数据指标中删除所有子级。 这些使子级的缓存无效。

应该有一种用默认值初始化给定 Child 的方法，通常只需调用`labels()`即可。 没有标签的指标必须始终被初始化，以避免缺少指标的问题。

### 数据指标名称 <a href="#metric-names" id="metric-names"></a>

数据指标名称必须遵循[规范](../concepts/data\_model.md#metric-names-and-labels)。与标签名称一样，对于 Gauge/Counter/Summary/Histogram 以及客户端库提供的任何其他收集器，都必须满足此要求。

许多客户端库提供了三个部分的名称设置：`namespace_subsystem_name`，其中只有`name`是必需的。

除了自定义收集器正在从其他集成/监控系统代理的情况下，必须不要使用动态/生成的数据指标名称或度量标准名称的子部分。 生成的/动态的数据指标名称表明您应该使用标签。

### 数据指标描述和帮助 <a href="#metric-description-and-help" id="metric-description-and-help"></a>

Gauge/Counter/Summary/Histogram 类型必须要求提供数据指标的帮助/描述。

客户端库随附的任何自定义收集器都必须具有关于其数据指标的描述/帮助。

建议将其设为必选参数，但不要检查它是否具有一定长度，就好像某人确实不想编写文档一样，我们也不会说服他们。 库(以及我们在生态系统中我们所能做到的)提供的收集器应具有良好的数据指标描述，以身作则。

## 公开数据 <a href="#exposition" id="exposition"></a>

客户端必须实现[公开格式](exposition\_formats.md)文档中描述的基于文本的导出格式

如果可以在不花费大量资源成本的情况下暴露数据指标，则可重现公开指标的顺序(尤其是对于人类可读格式)。

## 标准和运行时收集器 <a href="#standard-and-runtime-collectors" id="standard-and-runtime-collectors"></a>

客户端库应该提供标准导出的功能，如下所述。

以下这些应作为自定义收集器实现，并默认在默认的`CollectorRegistry`上注册。 应该有一个禁用这些功能的方法，因为在某些非常特殊的用例中，它们会受到阻碍。

### 进程数据指标 <a href="#process-metrics" id="process-metrics"></a>

这些指标的前缀为`process_`。如果所使用的语言或运行时获取必要的值是有问题的，甚至是不可能的，则客户端库应该宁愿忽略相应的数据指标，而不要输出伪造的、不准确的或特殊的值(如`NaN`)。 所有内存值以字节为单位，所有时间以 unixtime/seconds 为单位。

|               数据指标名称               |        含义       |   单位  |
| :--------------------------------: | :-------------: | :---: |
|     `process_cpu_seconds_total`    | 用户和系统 CPU 花费的时间 |   秒   |
|         `process_open_fds`         |    打开的文件描述符数量   | 文件描述符 |
|          `process_max_fds`         |     打开描述符最大值    | 文件描述符 |
|   `process_virtual_memory_bytes`   |      虚拟内存大小     |   字节  |
| `process_virtual_memory_max_bytes` |    最大可用虚拟内存量    |   字节  |
|   `process_resident_memory_bytes`  |      驻留内存大小     |   字节  |
|        `process_heap_bytes`        |   进程 heap 堆大小   |   字节  |
|    `process_start_time_seconds`    |  进程运行的 Unix 时间  |   秒   |

### 运行时数据指标 <a href="#runtime-metrics" id="runtime-metrics"></a>

客户端库还提供适当的前缀(如`go_`, `hotspot_`等)，以提供针对其语言运行时的指标(例如垃圾收集统计信息)等有意义的任何内容。

## 单元测试 <a href="#unit-tests" id="unit-tests"></a>

客户端库应该有包含核心集成库和数据暴露相关的的单元测试。

客户端库鼓励提供方便用户对其使用集成代码进行单元测试的方法。例如，Python 中的`CollectorRegistry.get_sample_value`。

## 打包和依赖 <a href="#packaging-and-dependencies" id="packaging-and-dependencies"></a>

理想情况下，客户端库可以包含在任何应用程序中，以在不破坏应用程序的情况下添加一些工具。

因此，在向客户端库添加依赖项时，建议谨慎。例如如果要添加的库使用的 Prometheus 客户端需要 x.y 版本的库，而应用程序在其他地方使用 x.z，这会对应用程序产生不利影响吗？

建议在可能出现这种情形的情况下，将核心工具与给定格式的数据指标的桥梁/暴露分开。例如，Java 示例 `simpleclient` 模块不具有依赖项，而`simpleclient_servlet`具有 HTTP 块。

## 性能注意事项 <a href="#performance-considerations" id="performance-considerations"></a>

客户端库必须是线程安全的，因此需要某种形式的并发控制，并且必须考虑多核计算机和应用程序的性能。

根据我们的经验，性能最差的是互斥对象。

处理器原子指令通常位于中间，且通常可以接受。

避免不通的 CPU 改变相同的 RAM 位的方法最有效，比如 Java 示例客户端中的 DoubleAdder。虽然有内存消耗。

如上所述，`labels()`的结果应该是可缓存的。倾向于使用标签进行度量筛选的并发映射往往相对较慢。没有标签的特殊外壳数据指标可以避免类似于`labels()`的查找，这会很有帮助

指标在增加/减少/设置等时应避免阻塞，因为在采集过程中不希望整个应用程序被阻塞。

鼓励对主要集成操作(包括标签)进行基准测试。

进行数据暴露时应牢记资源消耗，尤其是 RAM。考虑通过流式传输结果减少内存占用量，并可能限制并发采集数。
