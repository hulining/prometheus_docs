# 编写数据导出器

如果您要检测自己的代码，应遵循[有关如何使用 Prometheus 客户端库进行测试的一般规则](../practices/instrumentation.md)。从另外的监控系统或仪表系统获取数据指标时，事情往往不会那么简单。

本文档包含编写导出器或自定义收集器时应考虑的事项。涉及的理论也将对那些直接进行检测的人感兴趣。

如果您正在编写数据导出器，且不清楚此处的内容，请通过 IRC(Freenode 上的 #prometheus)或[邮件列表](https://prometheus.io/community)与我们联系。

## 可维护性和精简 <a href="#maintainability-and-purity" id="maintainability-and-purity"></a>

编写导出器时，您需要做出的主要决定是，您愿意投入多少工作来获取完善的指标。

如果有问题的系统只有少数的数据指标，那么所有事情都变得完美是一个好的选择。[HAProxy exporter](https://github.com/prometheus/haproxy\_exporter) 就是一个很好的例子。

另一方面，如果您尝试在系统具有数百个随新版本而频繁变化的指标时使事情变得完美，那么您已经注册了很多正在进行的工作。[MySQL exporter](https://github.com/prometheus/mysqld\_exporter) 位于此范围的末端。

[Node exporter](https://github.com/prometheus/node\_exporter) 是这些的混合体，其复杂度应模块而异。例如，`mdadm`收集器手动解析文件并暴露为此收集器创建的指标，因此我们也可以正确获取数据指标。对于`meminfo`收集器，其结果随内核版本的不同而不同，我们仅进行了充分的转换即可创建合法的指标。

## 配置 <a href="#configuration" id="configuration"></a>

在使用应用程序期间，你的目标应该是不需要自定义配置的数据导出器，而无需告诉应用程序在哪里。如果某些指标在大型架构中可能过于精细或代价过高，则可能还需要提供过滤这些指标的功能。例如，[HAProxy exporter](https://github.com/prometheus/haproxy\_exporter) 允许过滤每个服务器的统计信息。同样，默认情况下可能会禁用某些消耗过高的指标。

与其它监控系统，框架和协议配置使用时，您通常需要提供额外的配置或自定义内容以生成适用于 Prometheus 的数据指标。在最好的情况下，监控系统具有与 Prometheus 相似的数据模型，您可以自动确认如何转换指标。[Cloudwatch](https://github.com/prometheus/cloudwatch\_exporter), [SNMP](https://github.com/prometheus/snmp\_exporter) 和 [collectd](https://github.com/prometheus/collectd\_exporter) 就是这种情况。大多数情况，我们需要能够让用户选择他们想要提取的指标的能力。

在其它情况下，取决于系统和基础应用程序的使用情况，来自系统的数据指标完全是非标准的。在这种情况下，用户必须告诉我们如何转换指标。[JMX exporter](https://github.com/prometheus/jmx\_exporter) 是一个最典型的示例，[Graphite](https://github.com/prometheus/graphite\_exporter) 和 [StatsD](https://github.com/prometheus/statsd\_exporter) 数据导出器也需要配置来提取标签。

确保数据导出器无需配置即可开箱即用，并建议在需要时提供一些示例配置供转换。

YAML 是标准的 Prometheus 配置格式，默认情况下所有配置都应使用 YAML。

## 数据指标 <a href="#metrics" id="metrics"></a>

### 命名 <a href="#naming" id="naming"></a>

遵循[指标命名的最佳实践](../practices/naming.md)。

通常，数据指标名称应允许熟悉 Prometheus 而不是特定系统的人对数据指标的含义作出很好的预测。名为`http_requests_total`的数据指标不是很有用 - 这些数据是在某些过滤器中输入的，还是在到达用户代码时被测量的？`requests_total`甚至更糟，究竟是什么类型的请求？

使用直接检测，给定的数据指标应恰好在一个文件中。因此，在导出器和收集器中，一个数据指标应恰好适用于一个子系统并进行相应的命名。

除了编写自定义收集器或导出器外，永远不要以程序方式生成度量标准名称。

应用程序的数据指标名称通常应以出口者名称为前缀，例如`haproxy_up`

数据指标必须使用基本单位(如，秒，字节)，并可以将他们转换为图形工具更易读的内容。无论最终使用什么单位，数据指标名称中的单位都必须与使用中的单位匹配。同样，公开比率，还是不是百分比。更好的是，为比率两个分量中的每个分量指定一个计数器。

数据指标名称不应包含导出时所使用的标签，如`by_type`,因为如果标签被汇总，它将毫无意义。

例外是当您通过多个数据指标暴露带有不同标签的相同数据时，通常这是区分它们的最好方法。对于直接检测，仅当导出带有所有标签的单个数据指标的基础过高时，才应该出现这种情况。

Prometheus 数据指标和标签名称以`snake_case`(蛇形命名法)格式编写。将`camelCase`(驼峰命名法)转换为`snake_case`是希望的，由于自动执行转换操作并不总是会对像`myTCPExample`或`isNaN`之类的名称产生好的结果，因此有时最好保持原样

暴露的数据指标不应包含冒号，这些是为聚合时使用的用户自定义的记录规则保留的。

数据指标名称中仅能包含`[a-zA-Z0-9:_]`字符，任何其它字符都应该转化为下划线。

`_sum`, `_count`, `_bucket`和`_total`后缀被使用在 Summaries, Histograms 和 Counters 指标类型中。除非您要生成其中之一，否则避免使用这些后缀。

`_total`是 Counter 指标类型的公约。如果您使用的是计数器类型，就应该使用它。

保留`process_`和`scrape_`前缀。如果它们遵循 [matching semantics](https://docs.google.com/document/d/1Q0MXWdwp1mdXCzNRak6bW5LLVylVRXhdi7\_21Sg15xQ/edit)，则可以在这些字段上上添加自己的前缀上。例如，Prometheus 有`scrape_duration_seconds`来获取采集所需时间。类似的，以数据导出器为中心的指标也是一个很好的做法，如`jmx_scrape_duration_seconds`表示指定导出器花费多长时间来完成其任务。对于访问 PID 进程的统计信息，Go 和 Python 都提供了收集器来为您处理。这方面的一个很好的例子是[HAProxy exporter](https://github.com/prometheus/haproxy\_exporter)。

当您有成功的请求计数和失败的请求计数时，最好的公开方法是一个数据指标是总请求数据指标的另一个数据指标是失败请求指标。这使得计算失败率变得容易。不要使用带有失败或成功标签的数据指标。同样的，对于缓存，命中或未命中，最好有一个指标作为总指标，另一个指标为命中指标。

考虑到使用监控的人对数据指标名称进行代码或网络搜索的可能性。如果名称非常完善且不太可能在使用这些名称的人之外使用(如 SNMP 和网络工程师之间)，那么将其保留原样可能是个好主意。这种逻辑并不适用于所有数据导出器，如，MySQL 导出器导出的数据指标可能被各种各样的人使用，而不仅仅是 DBA。具有原始名称的`HELP`字符串可以提供与使用原始名称相同的大多数好处。

### 标签 <a href="#labels" id="labels"></a>

请阅读关于标签的[一些建议](../practices/instrumentation.md#things-to-watch-out-for)

避免将`type`作为标签名称，因为它太笼统，而且通常没有意义。您还应该尽可能避免使用可能与目标标签冲突的名称，如`region`, `zone`, `cluster`, `availability_zone`, `az`, `datacenter`, `dc`, `owner`, `customer`, `stage`, `service`, `environment`和`env`。但是，如果这就是应用程序所调用的某些资源，则最好不要通过重命名而引起混乱。

避免仅因为它们共享前缀就将它们放入一个度量标准的诱惑。除非您确定某一项指标有意义，否则多个指标会更安全。标签`le`对于 Histogram 具有特殊含义，而`quantile`对于 Summary有特殊含义。通常避免使用这些标签。

最好将读/写、发送/接收作为单独的数据指标，而不是作为标签。这通常是因为您一次只关心其中一个，且以单独的数据指标使用它们更容易。

经验法则是，一个指标在求和或求平均值时应该有意义。数据导出器还存在另一种情况，那就是数据从根本上是表格格式的，因此这将要求用户对数据指标名称进行正则表达式匹配才能使用。考虑一下主板上的电压传感器，尽管对它们进行数学运算是没有意义的，但将他们设置为一个数据指标而不是每个传感器具有单独的数据指标是很有意义的。数据指标内的所有值都应始终具有相同的单位，例如，请考虑是否将风扇速度与电压混合在一起，并且您无法自动将他们分开。

请不要这么做:

```
my_metric{label=a} 1
my_metric{label=b} 6
my_metric{label=total} 7
```

或这么做

```
my_metric{label=a} 1
my_metric{label=b} 6
my_metric{} 7
```

前者会误导对您的数据指标执行`sum()`的人，而后者会中断求和且很难使用。某些客户端库(如 Go)会主动尝试阻止您在自定义收集器中执行后者，而所有客户端库都应该您使用直接检测来执行后者。请绝对不要做其中任何一个，而是依靠 Prometheus 内置的聚合。

如果您的监控显示了这样的总数，请删除相关总指标。如果您出于某种原因不得不保留它，例如，总数包括未单独计数的事物，请使用不同的数据指标名称。

仪器标签应尽量少，每增加一个标签，用户在编写 promQL 时就需要多考虑一个标签。因此，请避免使用可以在不影响时间序列唯一性的情况下删除的仪器标签。可以通过信息指标添加有关指标的其它信息，例如，请参加如下的示例，了解如何处理版本号。

但是，在某些情况下，预计几乎所有用户指标都需要附加信息。如果是这样，添加非唯一标签而不是信息指标是正确的解决方案。例如,[mysqld\_exporter](https://github.com/prometheus/mysqld\_exporter)的`mysqld_perf_schema_events_statements_tota`l的`digest`标签是完整查询模式的哈希，并且对于唯一性而言已足够。但是，除了可读性，`digest_text`就用处较小了，因为对于长查询，该标签仅包含查询模式的开头且不是唯一的。因此我们为人性化保留`digest_text`标签并为唯一性保留`digest`标签。

### 目标标签，非静态采集标签 <a href="#target-labels-not-static-scraped-labels" id="target-labels-not-static-scraped-labels"></a>

如果您发现自己想对所有数据指标应用相同的标签，请停止。

通常有两种情况。

首先是对于某些标签将其包含在指标上很有用。例如，软件的版本号。相反，请使用[https://www.robustperception.io/how-to-have-labels-for-machine-roles/](https://www.robustperception.io/how-to-have-labels-for-machine-roles/)中描述的方法。

第二种情况是标签实际上是目标标签，例如，region, cluster names 等，它们来自基础结构设置而不是应用程序本身。并不是说应用程序适合您的标签分类，而是让运行 Prometheus 服务的人员进行配置，并且监控同一应用程序的不同人员可能为其赋予不同的名称。

因此，无论您使用哪种服务发现，这些标签都属于 Prometheus 的采集配置。也可以在这里应用机器角色的概念，因为对于至少某些人来说，这可能是有用的信息。

### 类型 <a href="#types" id="types"></a>

您应该尝试将您的指标类型与 Prometheus 类型进行匹配。这通常表示 Counter 和 Gauges。Summary 的`_count`和`_sum`也相对常见，有时您会看到 quantiles。Histogram 是罕见的，如果您遇到 Histogram，请记住，曝光格式会显示累积值。

通常，指标类型并不明显，尤其是当您自动处理一组指标时。通常，`UNTYPED`是安全的默认设置。

Counter 的值不会下降，因此，如果您的计数器类型来自另一个可以递减的检测系统，例如 Dropwizard 指标，则它不是 Counter 类型，而是 Gauge 类型。`UNTYPED`可能是最好的类型，因为如果将`GAUGE`用作计数器，则会产生误导。

### 帮助字符 <a href="#help-strings" id="help-strings"></a>

当您转换指标时，用户能够追溯到原始内容以及执行转换的规则是很有用的。将收集器或导出器的名称、所应用的任何规则的 ID 和原始度量的名称和详细信息放入帮助字符串中，将极大地帮助用户。

Prometheus 不喜欢具有不同帮助字符串的指标。如果您要从多个指标中选择一个指标，请选择其中一个指标以放入帮助字符串。

例如，SNMP exporter 使用 OID，JMX exporter 输入实例 mBean 名称。[HAProxy exporter](https://github.com/prometheus/haproxy\_exporter) 具有手写字符串。[node exporter](https://github.com/prometheus/node\_exporter) 也有各种各样的示例。

### 删除用处较小的统计数据 <a href="#drop-less-useful-statistics" id="drop-less-useful-statistics"></a>

除最小，最大和标准差外，某些监测系统还公开了 1m, 5m, 15m 速率，自应用启动以来的平均比率(例如，在 Dropwizard 数据指标中称为`mean`)。

这些都应该被删除，因为它们不是很有用，并且会增加混乱。Prometheus 自身可以计算速率且更为准确，因为暴露的平均值通常呈指数衰减。您不知道计算最小值或最大值的时间，标准偏差在统计上是无用的，并且如果需要计算平方和，你可以使用`_sum`和`_count`计算它。

Quantiles 有相关的问题，您可以选择删除或将其放在 Summary 中。

### 以点分割的字符串 <a href="#dotted-strings" id="dotted-strings"></a>

许多监控系统没有标签，而是采用`my.class.path.mymetric.labelvalue1.labelvalue2.labelvalue3`的方式。

[Graphite](https://github.com/prometheus/graphite\_exporter) 和 [StatsD](https://github.com/prometheus/statsd\_exporter) 导出器共享一种使用小配置语言进行转换的方法。其他导出器应实施相同的规定，并受益于将其分开为一个单独的库。

## Collectors

为数据导出器实现收集器时，请勿使用常规的直接检测方法，然后在每个采集对象上更新指标。

而是每次都创建新的指标。在 Go 中，这通过`Collect()`方法中的 [MustNewConstMetric](https://godoc.org/github.com/prometheus/client\_golang/prometheus#MustNewConstMetric) 完成的。对于 Python，参见[https://github.com/prometheus/client\_python#custom-collectors](https://github.com/prometheus/client\_python#custom-collectors)。在 Java 中，在 collect 方法中，生成一个`List<MetricFamilySamples>`，参见 [StandardExports.java](https://github.com/prometheus/client\_java/blob/master/simpleclient\_hotspot/src/main/java/io/prometheus/client/hotspot/StandardExports.java) 示例

其原因有两个。首先，可能会同时发生两次采集，直接检测使用的是有效的全局变量，因此您可能会发生竞争。其次，如果标签值消失了，它仍然会被导出。

通过直接检测对导出器本身进行监测，例如导出程序在所有采集过程中传输的总字节数或执行的调用总数。对于不与单个目标绑定的导出器，例如 [blackbox exporter](https://github.com/prometheus/blackbox\_exporter) 和 [SMNP exporter](https://github.com/prometheus/snmp\_exporter)，这些导出器仅应在有效的`/metrics`调用中暴露，而不是在特定采集目标中暴露数据。

### 采集自身数据指标 <a href="#metrics-about-the-scrape-itself" id="metrics-about-the-scrape-itself"></a>

有时，您希望导出有关采集的数据指标，例如花费了多长时间或处理了多少条记录。

这些应该作为 Gauges 类型指标暴露例，因为他们是关于时间，采集，和以导出器名称为前缀的指标名称。如，`jmx_scrape_duration_seconds`。通常，`_exporter`被排除在外，如果导出器也可以用作收集器，则一定要排除它。

### 设备和进程指标 <a href="#machine-and-process-metrics" id="machine-and-process-metrics"></a>

许多系统(如，Elasticsearch)都暴露如 CPU、内存和文件系统信息等设备数据指标。当节点导出器在 Prometheus 生态系统中提供这些功能时，应删除此类指标。

在 Java 中，需要监测框架都暴露进程级别和 JVM 级别的统计信息，例如 CPU 和 GC。Java 客户端和 JMX 导出器已经通过 [DefaultExports.java](https://github.com/prometheus/client\_java/blob/master/simpleclient\_hotspot/src/main/java/io/prometheus/client/hotspot/DefaultExports.java) 已更好的形式包含了它们，因此也应该删除它们。

其它语言和框架也是类似的。

## 部署 <a href="#deployment" id="deployment"></a>

每个导出器都应该准确地监控一个应用程序实例，最好直接部署在同一台机器上。这意味着对于您运行的每个 HAProxy，都要运行`haproxy_exporter`进程。 对于每台运行 Mesos worker 进程的设备，您都在其上运行 [Mesos exporter](https://github.com/mesosphere/mesos\_exporter)，如果 master 也运行在该设备上，则需要另外启动一个 Mesos exporter。

其背后的理论是，对于直接检测，这就是您要做的，并且我们正在尝试与其他布局尽可能接近。这意味着所有服务发现都在 Prometheus 中完成，而不是在导出器完成。这还有一个好处，就是 Prometheus 拥有允许用户使用 [blackbox exporter](https://github.com/prometheus/blackbox\_exporter) 探查您的服务所需的目标信息。

但是有两个例外:

第一个是便随着应用程序运行时，您的监控完全没有意义。SNMP, blackbox 和 IPMI exporters 是其中的主要示例。IPMI 和 SNMP exporters 通常是黑盒，无法在其上运行代码(尽管如果可以在它们上运行 node exporter，那会更好)，且 blackbox exporter 在其中监控诸如 DNS名称，也没有任何内容可运行。在这种情况下，Prometheus 仍应进行服务发现，并将要进行数据采集的目标传递出去。有关示例，请参见 blackbox 和 SNMP exporters。

请注意，目前只有使用Go，Python 和 Java 客户端库编写这种类型的导出程序。

第二个例外是，您要从系统的随机实例中提取某些统计信息，而不必在意与您交互的是哪个。考虑要对数据运行一些业务查询然后导出一组MySQL副本。让导出器使用常规负载平衡方法与您的一个副本进行通信是最明智的方法。

当您监控具有 master 功能的系统时，此方法不适用，在这种情况下，您应该分别监控每个实例并在 Prometheus 中处理 "masterness(主节点的)"。这是因为并非总是只有一个主节点且改变 Prometheus 监控的目标将是很奇怪的。

### 调度 <a href="#scheduling" id="scheduling"></a>

仅当 Prometheus 采集数据指标时才将其从应用程序中获取，导出器不应根据自己的计时器执行数据采集操作。也就是说，所有采集动作应该是同步的。

因此，您不应在公开的数据指标上设置时间戳，而是让 Prometheus 负责它。如果您需要时间戳标记，则可能需要使用 [Pushgateway](pushing.md)。

如果某个数据指标的检索成本特别高，如超过一分钟，则可以缓存该指标。这应该在`HELP`字符串中注明。

Prometheus 的默认采集超时时间为 10 秒。如果您的导出器采集时间预计会超过此值，则应在用户文档中明确指出这一点。

### 推送 <a href="#pushes" id="pushes"></a>

某些应用程序和监控系统仅推送数据指标，例如 StatsD, Graphite 和 collectd。

这里有两个注意事项。

首先，您何时将指标标记为过期？Collectd 以及与Graphite 进行通信的工具都定期导出，并且当它们停止时，我们希望停止公开指标。Collectd 包含有效期，因此我们可以使用它，而 Graphite 则不是，所以它(是否过期)是在导出器上的一个标志。

StatsD 有点不同，因为它处理事件而不是指标。最好的模型是在每个应用程序并行运行一个导出器，并在应用程序重新启动时重新启动它们，以便清除状态。

对于服务级别的指标，例如服务级别的批处理作业，您应该让导出器推入 Pushgateway 并在事件发生后退出，而不是自己处理状态。对于实例级批量指标，尚无明确的模式。这些选项要么是滥用节点导出器的文本文件收集器，要么依赖于内存中的状态(如果您不需要在重新启动后继续运行，则可能是最好的选择)或者实现与文本文件收集器类似的功能。

### 采集失败 <a href="#failed-scrapes" id="failed-scrapes"></a>

目前，采集失败有两种模式，您正在与之通信应用程序没有响应或出现其他问题。

第一种是返回 5xx 错误。

第二个使用一个`myexporter_up`指标，例如`haproxy_up`，它是值为 0 或 1变量的，具体取决于采集是否有效。

后者比较好，即使采集失败，您仍然可以获得一些有用的指标，例如提供过程统计信息的`HAProxy exporter`。前者对用户来说比较容易，因为`up`表示按常规方式可以正常运行，不过您无法区分是 exporter 还是应用程序处于关闭状态。

### 登陆页面 <a href="#landing-page" id="landing-page"></a>

如果用户访问`http://yourexporter/`具有一个带有导出器的名称以及指向`/metrics`页面的链接的简单 HTML 页面，那么对用户来说更好。

### 端口号 <a href="#port-numbers" id="port-numbers"></a>

用户可能在同一台计算机上有许多 exporter 和 Prometheus 组件，因此，使每个 exporter 和 Prometheus 组件具有唯一的端口号会使部署变得更加容易。

[https://github.com/prometheus/prometheus/wiki/Default-port-allocations](https://github.com/prometheus/prometheus/wiki/Default-port-allocations) 是我们跟踪记录它们的地方，这是可公开编辑的。

开发 exporter 时，最好在发布之前，获取下一个可用的端口号。如果您尚未准备好发布，您也可以保留您的用户名和 WIP(半成品)。

这是一个注册表，旨在使我们的用户的使用更加轻松，而不是开发指定 exporter 的提交仓库。 对于内部应用程序的 exporter，我们建议使用已经默认分配的端口范围之外的端口。

## 发布 <a href="#announcing" id="announcing"></a>

准备好向公众发布您的 exporter 后，请向邮件列表发送电子邮件并提交 PR，以将其添加到[可用的 exporter 列表](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exporters.md)中。
