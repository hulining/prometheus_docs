---
title: 存储
---

# 存储

Prometheus 包括本地磁盘时间序列数据库，也可以选择与远程存储系统集成。

## 本地存储 <a id="local-storage"></a>

Prometheus 的本地时间序列数据库以自定义格式在磁盘上存储时间序列。

### 磁盘设计 <a id="on-disk-layout"></a>

存储的数据样本每两小时分为一个块。每两小时的块由一个目录组成，该目录包含一个或多个块文件，该文件包含该时间窗口的所有时间序列样本，以及元数据文件和索引文件\(用于将数据指标名称和标签索引到块文件中的时间序列\)。通过 API 删除时间序列时，删除记录存储在单独的逻辑删除文件中\(而不是立即从块文件中删除\)。

当前传入的时间序列样本保留在内存中，尚未完全保留。它通过于预写日志\(WAL\)防止崩溃，当 Prometheus 服务在崩溃后重新启动时，可以重新重放该日志。预写日志文件以 128MB 的段存储在 wal 目录中。这些文件包含尚未压缩的原始数据，因此它们比常规的块文件大得多。Prometheus 至少保留 3 个预写日志文件，但是高流量服务器可能会看到三个以上的 WAL 文件，因为它至少需要保留两个小时的原始数据。

Prometheus 服务的数据目录的目录结构如下所示：

```text
./data
├── 01BKGV7JBM69T2G1BGBGM6KB12
│   └── meta.json
├── 01BKGTZQ1SYQJTR4PB43C8PD98
│   ├── chunks
│   │   └── 000001
│   ├── tombstones
│   ├── index
│   └── meta.json
├── 01BKGTZQ1HHWHV8FBJXW1Y3W0K
│   └── meta.json
├── 01BKGV7JC0RY8A6MACW02A2PJD
│   ├── chunks
│   │   └── 000001
│   ├── tombstones
│   ├── index
│   └── meta.json
└── wal
    ├── 00000002
    └── checkpoint.000001
```

请注意，本地存储的局限性在于它不是集群的且没有副本。因此，对于磁盘或节点发生故障，它不是可任意伸缩或可持久化的，应该像其它类型的单节点数据库一样对待它。建议使用 RAID 来提高磁盘可用性，并用[快照](querying/api.md#snapshot)进行备份，容量规划等，以提高可用性。通过适当的存储可用性和计划，可以在本地存储中存储多年的数据。

另外，可以通过[远程读/写 API](../operating/integrations.md#remote-endpoints-and-storage) 使用外部存储。这些系统的可用性，性能和效率差异很大，因此需要仔细评估。

有关文件格式的更多详细信息，参见 [TSDB 格式](https://github.com/prometheus/prometheus/blob/master/tsdb/docs/format/README.md)

## 压缩 <a id="compaction"></a>

最初两个小时的块最终在后台压缩为更长的块。

压缩将创建较大的块，最多保留时间的 10% 或 31天，以较小者为准。

## 操作细节 <a id="operational-aspects"></a>

Prometheus 有几个允许配置本地存储的标志。其中最重要的几个是：

* `--storage.tsdb.path`: 指定 Prometheus 在何处写入数据库\(数据库保存位置\)。默认为`data/`
* `--storage.tsdb.retention.time`: 指定何时删除旧数据\(旧数据保存时间\)。默认为`15d`。如果此标志位设置为默认值之外的其它值，则覆盖`storage.tsdb.retention`
* `--storage.tsdb.retention.size`: \[实验性的\]存储块可以使用的最大字节数\(请注意，这不包括 WAL 的大小,这可能是很大的\)。最旧的数据首先被删除，默认为`0`或被禁用。该标识是实验性的，在将来的版本中可能进行更改。支持的单位为`KB, MB, GB, PB.`
* `--storage.tsdb.retention`: 该标志位已被`storage.tsdb.retention.time`标志位代替
* `--storage.tsdb.wal-compression`: 该标志位启用预写日志\(WAL\)的压缩。取决于您的数据，可以将预期的 WAL 大小减少一半，而额外的 CPU 负载却很少。请注意，如果启用此标志，Prometheus 降级到 2.11.0 以下版本时，您将需要删除 WAL，因为它会变的不可读。

平均下来，Prometheus 每个样本仅使用大约 1-2 个字节。因此要计划 Prometheus 服务的所需要的容量，可以粗略的使用以下公式计算：

```text
needed_disk_space = retention_time_seconds * ingested_samples_per_second * bytes_per_sample
```

要调整每秒保存样本的速率，可以减少采集的时间序列数\(减少目标或减少目标采集序列\)或者增加采集数据的时间间隔。由于在一个序列中的样本进行压缩，所以减少序列数可能会更有效。

如果您的本地存储由于某种原因损坏，最好的选择是关闭 Prometheus 并删除整个存储目录。Prometheus 的本地存储不支持不兼容 POSIX 的文件系统，可能损坏，无法恢复。NFS 仅是潜在的 POSIX，大多数实现不是。您可以尝试删除单个块目录来解决该问题，这意味着每个块目录损失的时间窗口大约为两个小时。同样，Prometheus 的本地存储并不意味着持久的长期存储。

如果同时指定了时间和文件大小保留策略，将使用第一个触发的策略。

清除过期的块将在后台运行。删除过期的块可能最多需要两个小时，过期的块在清除之前必须完全过期。

## 远程存储集成 <a id="remote-storage-integrations"></a>

Prometheus 的本地存储在可伸缩性和持久性方面受到单个节点的限制。Prometheus 并没有解决 Prometheus 本身的集群存储，而是提供了一组允许与远程存储系统集成的接口。

### 总览 <a id="overview"></a>

Prometheus 通过两种方式与远程存储系统集成：

* Prometheus 可以将采集的数据样本以标准格式写入远程 URL。
* Prometheus 可以以标准格式从远程 URL 读取\(返回\)样本数据

![remote\_integrations](https://prometheus.io/docs/prometheus/2.17/images/remote_integrations.png)

读写协议都使用基于 HTTP 的快速压缩协议缓冲区编码。该协议尚未被认为是稳定的API，当 Prometheus 和远程存储之间的所有跃点都支持 HTTP/2 时，该协议可能会更改为在 HTTP/2 上使用 gRPC。

有关在 Prometheus 中配置远程存储集成的详细信息，请参阅 Prometheus 配置文档的[远程写入](configuration/configuration.md#remote_write)和[远程读取](configuration/configuration.md#remote_read)部分。

有关请求和响应消息的详细信息，请参阅[远程存储缓冲区协议定义](https://github.com/prometheus/prometheus/blob/master/prompb/remote.proto)。

请注意，在读取过程中，Prometheus 仅从远端存储获取一组由标签选择器和时间范围筛选的原始序列数据。PromQL 对原始数据的所有计算仍然在 Prometheus 本身中进行。这意味着远程读取查询具有一定的可伸缩性限制，因为所有必需的数据都需要先加载到查询的 Prometheus 服务中，然后再在其中进行处理。但是，暂时认为支持 PromQL 的完全分布式计算是不可行的。

### 现有集成 <a id="existing-integrations"></a>

要了解有关与现有远程存储系统集成的更多信息，请参阅[集成文档](../operating/integrations.md#remote-endpoints-and-storage)。

