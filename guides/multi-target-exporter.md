---
title: 理解并使用 multi-target exporters 模式
---

# 理解并使用 multi-target exporters 模式

本指南将向您介绍 multi-target exporters 模式。为此，我们将:

* 描述 multi-target exporters 模式及其使用原因。
* 运行 [blackbox](https://github.com/prometheus/blackbox_exporter) exporter 作为一个模式示例
* 为 blackbox exporter 配置自定义查询模块
* 让 blackbox exporter 针对 Prometheus [网站](https://prometheus.io)运行基本指标查询
* 研究配置 Prometheus 的流行模式以使用重新标记来采集 exporter 数据.

## 什么是 multi-target exporters 模式? <a id="the-multi-target-exporter-pattern"></a>

通过多目标[导出器](../instrumenting/exporters.md)模式，我们引用了一种特定的设计，其中：

* 导出器将通过网络协议获取目标的指标
* 导出器不必在获取数据指标的计算机上运行
* 导出器将获取目标和查询配置字符串作为 Prometheus 的 GET 请求的参数
* 导出器在收到 Prometheus 的 GET 请求后开始采集，并且在完成采集后开始。
* 导出器可以查询多个目标

此模式仅用于某些导出器，例如 [blackbox](https://github.com/prometheus/blackbox_exporter) 和 [SNMP exporter](https://github.com/prometheus/snmp_exporter).

原因是我们要么无法在目标上运行导出器，例如 网络设备使用 SNMP，或者我们对距离明确感兴趣，如来自我们网络之外的特定点网站的延迟和可访问性，这是[ blackbox exporter](https://github.com/prometheus/blackbox_exporter) 的常见用例。

## 运行 multi-target exporters <a id="running-multi-target-exporters"></a>

多目标出口商在运行环境方面具有灵活性，并且可以通过多种方式运行。作为常规程序，在容器中，作为后台服务，在裸机上，在虚拟机上等。因为要查询它们并通过网络进行查询，所以它们需要适当的开放端口。否则，它们是无用的。

现在让我们自己尝试一下

通过在终端中运行如下命令使用 [Docker](https://www.docker.com/) 启动 blackbox exporter 容器。根据您的系统配置，您可能需要在命令前加上`sudo`:

```bash
docker run -p 9115:9115 prom/blackbox-exporter
```

如果一切顺利，您应该看到一些日志行，最后一行应该包含 `msg="Listening on address"`，如下所示:

```text
level=info ts=2018-10-17T15:41:35.4997596Z caller=main.go:324 msg="Listening on address" address=:9115
```

## multi-target exporters 的基本查询 <a id="basic-querying-of-multi-target-exporters"></a>

有两种查询方式

1. exporter 本身查询。它有自己的指标，通常可以在 `/metrics` 进行访问.
2. 以采集另一个目标查询 exporter. 通常在 "配置的" 端点上可用, e.g. `/probe`。使用 multi-target exporters 时，这可能是您最感兴趣的

您可以在另一个终端中使用 curl 手动尝试第一种查询类型，或使用如下[链接](http://localhost:9115/metrics)

```bash
curl 'localhost:9115/metrics'
```

响应应该类似如下所示:

```text
# HELP blackbox_exporter_build_info A metric with a constant '1' value labeled by version, revision, branch, and goversion from which blackbox_exporter was built.
# TYPE blackbox_exporter_build_info gauge
blackbox_exporter_build_info{branch="HEAD",goversion="go1.10",revision="4a22506cf0cf139d9b2f9cde099f0012d9fcabde",version="0.12.0"} 1
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
go_gc_duration_seconds{quantile="0.25"} 0
go_gc_duration_seconds{quantile="0.5"} 0
go_gc_duration_seconds{quantile="0.75"} 0
go_gc_duration_seconds{quantile="1"} 0
go_gc_duration_seconds_sum 0
go_gc_duration_seconds_count 0
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 9

[…]

# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 0.05
# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1.048576e+06
# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 7
# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 7.8848e+06
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.54115492874e+09
# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 1.5609856e+07
```

这些数据指标遵循 Prometheus [格式](../instrumenting/exposition_formats.md#text-based-format). 他们来源于 exporter 的 [instrumentation](../practices/instrumentation.md),并告诉我们 exporter 运行时的状态。如果您感到好奇，请尝试一下我们的指南，以了解如何[安装自己的应用程序](go-application.md).

对于第二种查询，我们需要在 HTTP GET Request 中提供 target 和 module 作为参数。target 是一个 URL 或 IP，而 module 必须定义在 exporter 的配置中。blackbox exporter 容器带有一个有用的示例配置。

我们将使用 `prometheus.io` target 和 预定义的 `http_2xx` module。它告诉 exporter 像浏览器一样发出 GET 请求，如果您访问到 `prometheus.io` 并期望 [200 OK](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#2xx_Success) 响应。

现在，您可以在终端中使用 curl 告诉您的黑盒导出器访问 `prometheus.io`：

```bash
curl 'localhost:9115/probe?target=prometheus.io&module=http_2xx'
```

这将返回很多数据指标:

```text
# HELP probe_dns_lookup_time_seconds Returns the time taken for probe dns lookup in seconds
# TYPE probe_dns_lookup_time_seconds gauge
probe_dns_lookup_time_seconds 0.061087943
# HELP probe_duration_seconds Returns how long the probe took to complete in seconds
# TYPE probe_duration_seconds gauge
probe_duration_seconds 0.065580871
# HELP probe_failed_due_to_regex Indicates if probe failed due to regex
# TYPE probe_failed_due_to_regex gauge
probe_failed_due_to_regex 0
# HELP probe_http_content_length Length of http content response
# TYPE probe_http_content_length gauge
probe_http_content_length 0
# HELP probe_http_duration_seconds Duration of http request by phase, summed over all redirects
# TYPE probe_http_duration_seconds gauge
probe_http_duration_seconds{phase="connect"} 0
probe_http_duration_seconds{phase="processing"} 0
probe_http_duration_seconds{phase="resolve"} 0.061087943
probe_http_duration_seconds{phase="tls"} 0
probe_http_duration_seconds{phase="transfer"} 0
# HELP probe_http_redirects The number of redirects
# TYPE probe_http_redirects gauge
probe_http_redirects 0
# HELP probe_http_ssl Indicates if SSL was used for the final redirect
# TYPE probe_http_ssl gauge
probe_http_ssl 0
# HELP probe_http_status_code Response HTTP status code
# TYPE probe_http_status_code gauge
probe_http_status_code 0
# HELP probe_http_version Returns the version of HTTP of the probe response
# TYPE probe_http_version gauge
probe_http_version 0
# HELP probe_ip_protocol Specifies whether probe ip protocol is IP4 or IP6
# TYPE probe_ip_protocol gauge
probe_ip_protocol 6
# HELP probe_success Displays whether or not the probe was a success
# TYPE probe_success gauge
probe_success 0
```

请注意，几乎所有数据指标的值都为 `0`。最后一个记录为 `probe_success 0`。这意味着探针无法到达 `prometheus.io`。原因隐藏在值为 `6` 的 `probe_ip_protocol` 中。默认情况下，探针使用 [IPv6](https://en.wikipedia.org/wiki/IPv6) 除非另行设置。但是 Docker 守护程序会阻止 IPv6。因此，我们在 Docker 中运行的 blackbox exporter 容器无法通过IPv6连接。

我们可以设置 Docker 允许 IPv6 或 blackbox exporter 使用IPv4。实际上，这两者都是有意义的。因此，"该怎么办的"回答一般是"视情况而定"。因为这是 exporter 指南，我们将更换 exporter，并借此机会配置自定义模块\(module\)。

## 配置 modules <a id="configuring-modules"></a>

modules 是在 docker 容器内名为 `config.yml` 的文件中预定义的 `config.yml`，该文件是 github 代码库中 [blackbox.yml](https://github.com/prometheus/blackbox_exporter/blob/master/blackbox.yml) 的副本。ch is a copy of [blackbox.yml](https://github.com/prometheus/blackbox_exporter/blob/master/blackbox.yml) in the github repo.

我们将复制此文件，[配置](https://github.com/prometheus/blackbox_exporter/blob/master/CONFIGURATION.md) 为我们想要的配置，并让 exporter 使用我们的配置文件，而不是容器中默认包含的配置文件。

首先使用 curl 或浏览器下载文件：

```bash
curl -o blackbox.yml https://raw.githubusercontent.com/prometheus/blackbox_exporter/master/blackbox.yml
```

在编辑器中打开它。前几行如下所示

```yaml
modules:
  http_2xx:
    prober: http
  http_post_2xx:
    prober: http
    http:
      method: POST
```

[YAML](https://en.wikipedia.org/wiki/YAML) 使用空格缩进来表示层次结构，您可以看到定义了名为 `http_2xx` 和 `http_post_2xx` 的模块。且它们都具有 `http` 探测器，并且其中一个 method 的值专门设置为 `POST`。

现在，您可以通过将 `http` 探针的 `preferred_ip_protocol` 字段显式设置为字符串 `ip4` 来更改 `http_2xx` module。

```yaml
modules:
  http_2xx:
    prober: http
    http:
      preferred_ip_protocol: "ip4"
  http_post_2xx:
    prober: http
    http:
      method: POST
```

如果您想进一步了解可用的探测器和选项，请查看 [文档](https://github.com/prometheus/blackbox_exporter/blob/master/CONFIGURATION.md).

现在，我们需要告诉 blackbox exporter 使用我们新更改的文件。您可以使用标志 `--config.file="blackbox.yml"` 实现。但是由于我们使用的是 Docker，因此我们首先必须使用`--mount` 命令在容器内将此文件[设置为可用](https://docs.docker.com/storage/bind-mounts/)

NOTE: 如果您使用的是 macOS，则首先需要允许 Docker 守护程序访问 `blackbox.yml` 所在的目录。您可以通过单击菜单栏中的 Docker 小鲸鱼 logo ，然后单击 `Preferences`-&gt; `File Sharing`-&gt; `+` 来实现。然后点击 `Apply & Restart`。

首先，通过切换到旧容器终端按`ctrl+c` 停止旧容器。 确保您位于包含 `blackbox.yml` 的目录中。然后，您运行此命令。虽然很长，但我们会解释:

{\#run-exporter}

```bash
docker \
  run -p 9115:9115 \
  --mount type=bind,source="$(pwd)"/blackbox.yml,target=/blackbox.yml,readonly \
  prom/blackbox-exporter \
  --config.file="/blackbox.yml"
```

使用此命令，您告诉 `docker` 执行以下操作：

1. `run`\(运行\)一个容器，并将容器外部的 `9115` 端口映射到容器内部的 `9115` 端口.
2. 以 `readonly` 模式 `mount`\(挂载\)您当前目录下的\(`$(pwd)` 表示当前工作目录\) `blackbox.yml` 文件到 `/blackbox.yml`
3. 使用 [Docker hub](https://hub.docker.com/r/prom/blackbox-exporter/) 上的 `prom/blackbox-exporter` 镜像 .
4. 运行带有 `--config.file` 的 blackbox-exporter，并告诉它使用 `/blackbox.yml` 作为配置文件 

如果一切正常，您应该看到类似如下内容的输出:

```text
level=info ts=2018-10-19T12:40:51.650462756Z caller=main.go:213 msg="Starting blackbox_exporter" version="(version=0.12.0, branch=HEAD, revision=4a22506cf0cf139d9b2f9cde099f0012d9fcabde)"
level=info ts=2018-10-19T12:40:51.653357722Z caller=main.go:220 msg="Loaded config file"
level=info ts=2018-10-19T12:40:51.65349635Z caller=main.go:324 msg="Listening on address" address=:9115
```

现在，您可以在终端中尝试使用支持 IPv4 的新的 `http_2xx` module:

```bash
curl 'localhost:9115/probe?target=prometheus.io&module=http_2xx'
```

返回的 Prometheus 指标应如下所示:

```text
# HELP probe_dns_lookup_time_seconds Returns the time taken for probe dns lookup in seconds
# TYPE probe_dns_lookup_time_seconds gauge
probe_dns_lookup_time_seconds 0.02679421
# HELP probe_duration_seconds Returns how long the probe took to complete in seconds
# TYPE probe_duration_seconds gauge
probe_duration_seconds 0.461619124
# HELP probe_failed_due_to_regex Indicates if probe failed due to regex
# TYPE probe_failed_due_to_regex gauge
probe_failed_due_to_regex 0
# HELP probe_http_content_length Length of http content response
# TYPE probe_http_content_length gauge
probe_http_content_length -1
# HELP probe_http_duration_seconds Duration of http request by phase, summed over all redirects
# TYPE probe_http_duration_seconds gauge
probe_http_duration_seconds{phase="connect"} 0.062076202999999996
probe_http_duration_seconds{phase="processing"} 0.23481845699999998
probe_http_duration_seconds{phase="resolve"} 0.029594103
probe_http_duration_seconds{phase="tls"} 0.163420078
probe_http_duration_seconds{phase="transfer"} 0.002243199
# HELP probe_http_redirects The number of redirects
# TYPE probe_http_redirects gauge
probe_http_redirects 1
# HELP probe_http_ssl Indicates if SSL was used for the final redirect
# TYPE probe_http_ssl gauge
probe_http_ssl 1
# HELP probe_http_status_code Response HTTP status code
# TYPE probe_http_status_code gauge
probe_http_status_code 200
# HELP probe_http_uncompressed_body_length Length of uncompressed response body
# TYPE probe_http_uncompressed_body_length gauge
probe_http_uncompressed_body_length 14516
# HELP probe_http_version Returns the version of HTTP of the probe response
# TYPE probe_http_version gauge
probe_http_version 1.1
# HELP probe_ip_protocol Specifies whether probe ip protocol is IP4 or IP6
# TYPE probe_ip_protocol gauge
probe_ip_protocol 4
# HELP probe_ssl_earliest_cert_expiry Returns earliest SSL cert expiry in unixtime
# TYPE probe_ssl_earliest_cert_expiry gauge
probe_ssl_earliest_cert_expiry 1.581897599e+09
# HELP probe_success Displays whether or not the probe was a success
# TYPE probe_success gauge
probe_success 1
# HELP probe_tls_version_info Contains the TLS version used
# TYPE probe_tls_version_info gauge
probe_tls_version_info{version="TLS 1.3"} 1
```

你可以看到成功探测并获取到很多有用的数据指标，例如用 [Unix 时间](https://en.wikipedia.org/wiki/Unix_time) 表示的延迟，状态码，ssl 状态或证书的过期时间

Blackbox exporter 还在 [localhost:9115](http://localhost:9115) 提供了一个小型 Web 界面，供您检查最后几个探针、已加载的配置和调试信息。它甚至提供了到探针 `prometheus.io` 直接链接，如果您想知道为什么某些功能不起作用，则非常方便。

## 使用 Prometheus 查询多目标 exporters <a id="querying-multi-target-exporters-with-prometheus"></a>

到现在为止还好，恭喜您，Blackbox exporter 可以工作，您可以手动告诉它远程查询的目标。现在，您需要告诉 Prometheus 为我们进行查询。

下面，您可以看到一个最小的 Prometheus 配置。它设置 Prometheus 像我们之前使用 `curl 'localhost:9115/metrics'` 采集 exporter 本身:

NOTE: 如果您在 Mac 或 Windows 中使用 Docker，则最后一步不能使用 `localhost:9115`，而必须使用 `host.docker.internal:9115`。这与在这些操作系统上实现的虚拟机相关，您不应该在生产环境使用使用它。

`prometheus.yml` for Linux

```yaml
global:
  scrape_interval: 5s

scrape_configs:
- job_name: blackbox # To get metrics about the exporter itself
  metrics_path: /metrics
  static_configs:
    - targets:
      - localhost:9115
```

`prometheus.yml` for macOS and Windows:

```yaml
global:
  scrape_interval: 5s

scrape_configs:
- job_name: blackbox # To get metrics about the exporter itself
  metrics_path: /metrics
  static_configs:
    - targets:
      - host.docker.internal:9115
```

现在运行 Prometheus 容器，并告诉他挂载我们上述配置文件。由于可以通过主机寻址容器网络，因此您需要在 Linux 与 MacOS 和 Windows 上使用稍有不同的指令

{\#run-prometheus}

在 Linux 上运行 Prometheus\(不要在生产环境中使用 `--network="host"` 选项\):

```bash
docker \
  run --network="host"\
  --mount type=bind,source="$(pwd)"/prometheus.yml,target=/prometheus.yml,readonly \
  prom/prometheus \
  --config.file="/prometheus.yml"
```

在 MacOS 和 Windows 中运行 Prometheus:

```bash
docker \
  run -p 9090:9090 \
  --mount type=bind,source="$(pwd)"/prometheus.yml,target=/prometheus.yml,readonly \
  prom/prometheus \
  --config.file="/prometheus.yml"
```

该命令的运行方式类似于使用指定配置文件运行 blackbox exporter.

如果一切正常，您应该可以访问 [localhost:9090/targets](http://localhost:9090/targets) 并看到在 `blackbox` 下端点为绿色的 `UP` 。如果你看到一个红色的 `DOWN` 请确保您上面启动的 blackbox exporter 仍在运行 。如果您什么也没有看到，或者看到黄色的 `UNKNOWN`，则表示您需要等几秒钟的时间再刷新您浏览器的页面。

为了设置 Prometheus 查询 `"localhost:9115/probe?target=prometheus.io&module=http_2xx"` 您再添加另一个名为 `blackbox-http` 的采集作业，并在 Prometheus 配置文件 `prometheus.yml` 设置 `metrics_path` 为 `/probe`, `params` 为如下参数:

{\#prometheus-config}

```yaml
global:
  scrape_interval: 5s

scrape_configs:
- job_name: blackbox # To get metrics about the exporter itself
  metrics_path: /metrics
  static_configs:
    - targets:
      - localhost:9115   # For Windows and macOS replace with - host.docker.internal:9115

- job_name: blackbox-http # To get metrics about the exporter’s targets
  metrics_path: /probe
  params:
    module: [http_2xx]
    target: [prometheus.io]
  static_configs:
    - targets:
      - localhost:9115   # For Windows and macOS replace with - host.docker.internal:9115
```

保存配置文件后，切换到运行 Prometheus docker 容器的终端并使用 `ctrl+C` 停止它 `ctrl+C` 然后使用之前的命令再次启动它以便重新加载配置。

终端应返回 `"Server is ready to receive web requests."` 消息且几秒钟后，您可以在 [Prometheus](http://localhost:9090/graph?g0.range_input=5m&g0.stacked=0&g0.expr=probe_http_duration_seconds&g0.tab=0) 中看到彩色图表。

这可以工作，但有一些缺点

1. 实际目标位于参数配置中，这是非常不寻常的，以后很难理解.
2. `instance` 标签具有 blackbox exporter 地址的值，从技术上讲是真实的，但不是我们感兴趣的内容
3. 我们看不到我们探测的 URL。这是不切实际的，并且如果我们探查多个URL，也会将不同的指标混合为一个指标。

为了解决这个问题，我们将使用[重新标记](../prometheus/configuration/configuration.md#relabel_config). 重新标记在这里很有用，因为在幕后，Prometheus 中的许多东西都配置了内部标签。 详细信息很复杂，超出了本指南的范围。我们将说明限于必要的范围。如果您想了解更多信息，请查看此[演讲](https://www.youtube.com/watch?v=b5-SvvZ7AwI)。现在，只要您了解以下内容即可:

* 采集后，所有以 `__` 开头的标签都将被丢弃，大多数内部标签均以 `__` 开头
* 您可以设置名为 `__param_<name>` 的内部标签，那些使用 键 `<name>`为采集请求设置 URL 参数。
* 有一个内部标签 `__address__`，由 `static_configs` 下的 `target` 设置，其值是采集请求的主机名。默认情况下它稍后y用于设置 `instance` 标签的值，该值附加到每个指标，并告诉您指标来自哪里

这是您将用于执行此操作的配置。 不必担心一次过多，我们将逐步进行操作：

```yaml
global:
  scrape_interval: 5s

scrape_configs:
- job_name: blackbox # To get metrics about the exporter itself
  metrics_path: /metrics
  static_configs:
    - targets:
      - localhost:9115   # For Windows and macOS replace with - host.docker.internal:9115

- job_name: blackbox-http # To get metrics about the exporter’s targets
  metrics_path: /probe
  params:
    module: [http_2xx]
  static_configs:
    - targets:
      - http://prometheus.io    # Target to probe with http
      - https://prometheus.io   # Target to probe with https
      - http://example.com:8080 # Target to probe with http on port 8080
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: localhost:9115  # The blackbox exporter’s real hostname:port. For Windows and macOS replace with - host.docker.internal:9115
```

那么与上一个配置相比有什么新变化?

`params` 不在包含 `target`。相反，我们将实际目标添加到 `static configs: targets` 下。我们可以设置几个，因为我们现在可以这样做：

```yaml
  params:
    module: [http_2xx]
  static_configs:
    - targets:
      - http://prometheus.io    # Target to probe with http
      - https://prometheus.io   # Target to probe with https
      - http://example.com:8080 # Target to probe with http on port 8080
```

`relabel_configs` 包含重新标记的规则:

```yaml
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: localhost:9115  # The blackbox exporter’s real hostname:port. For Windows and macOS replace with - host.docker.internal:9115
```

在应用重新标记规则之前，Prometheus 发出的请求 URI 如下所示： `"http://prometheus.io/probe?module=http_2xx"`. 重新标记之后，它看起来像是这样的: `"http://localhost:9115/probe?target=http://prometheus.io&module=http_2xx"`.

现在让我们研究每个规则是如何工作的:

首先，我们从标签 `__address__` \(其中包含 `targets` 中的值\) 中获取值，并将它们写入新标签 `__param_target`，这将为 Prometheus 采集请求添加 `target` 参数:

```yaml
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
```

此后，我们想象中的 Prometheus 请求 URI 现在具有 target 参数: `"http://prometheus.io/probe?target=http://prometheus.io&module=http_2xx"`.

然后，我们从标签 `__param_target` 中获取值，并使用这些值创建 instance 标签。

```yaml
  relabel_configs:
    - source_labels: [__param_target]
      target_label: instance
```

我们的请求不会更改，但从我们的请求返回的数据指标将带有标签 `instance="http://prometheus.io"`.

之后，我们将 `__address__` 标签的值设置为 `localhost:9115`\(exporter 的 URI\)。这将用于 Prometheus 采集请求的主机名和端口，这样它就查询 exporter，而不是直接查询目标 URI。

```yaml
  relabel_configs:
    - target_label: __address__
      replacement: localhost:9115  # The blackbox exporter’s real hostname:port. For Windows and macOS replace with - host.docker.internal:9115
```

现在，我们的请求是 `"localhost:9115/probe?target=http://prometheus.io&module=http_2xx"`。这样，我们就可以在其中拥有实际的 target，并将它们作为 `instance` 标签值获取，同时让 Prometheus 向 blackbox exporter 发送请求

人们通常将这些与特定的服务发现结合在一起。查看相关[配置文档](../prometheus/configuration/configuration.md)获取更多信息。使用它们是没有问题的，因为写入 `__address__` 标签就像在 `static_config` 下定义 `target` 一样

这就对了，重新启动 Prometheus docker 容器并查看您的[数据指标](http://localhost:9090/graph?g0.range_input=30m&g0.stacked=0&g0.expr=probe_http_duration_seconds&g0.tab=0).请注意，您选择了实际收集指标的时间段。

## 小结 <a id="summary"></a>

在本指南中，您学习了多目标 exporter 模式是如何工作的，如何使用自定义模块运行 blackbox exporter，以及如何使用重新标记配置 Prometheus 来使用探针标签采集数据指标。

