---
title: 远程写调试
sort_rank: 8
---

# 远程写调试

Prometheus 为远程写入实现了合理的默认设置，但是用户有不同的要求，并且希望优化其远程设置。

本页介绍可通过[远程写入配置](../prometheus/configuration/configuration.md#remote_write)使用的调整参数。

## 远程写特征 <a id="remote-write-characteristics"></a>

每个远程写入目标都会启动一个从预写日志\(WAL\)中读取的队列，将样本写入到分片拥有的内存队列中，然后将请求发送到已配置的端点。数据流如下所示：

```text
      |-->  queue (shard_1)   --> remote endpoint
WAL --|-->  queue (shard_...) --> remote endpoint
      |-->  queue (shard_n)   --> remote endpoint
```

当一个分片备份并填满队列时，Prometheus 将阻止从 WAL 读取任何分片。除非远程端点保持关闭状态超过 2 小时，否则将重试失败而不会丢失数据。2 小时后，WAL 将被压缩，并且尚未发送的数据将丢失

在操作过程中，Prometheus 将根据传入的采样率，未发送的未处理样本数以及发送每个样本所花费的时间，不断计算要使用的最佳分片数

### 内存使用情况 <a id="memory-usage"></a>

使用远程写入会增加 Prometheus 的内存占用量。大多数用户报告的内存使用量增加了约 25%，但这取决于数据的类型。对于 WAL 中的每个序列，远程写代码都会缓存序列 ID 到标签值的映射，导致大量的序列搅动，从而显着增加内存使用率。

除了序列缓存之外，每个分片及其队列还会增加内存使用量。分片内存与`分片数量 * (capacity + max_samples_per_send)`成正比。进行调整时，请考虑减少`max_shards`，同时增加 `capacity` 和 `max_samples_per_send` 以避免无意中耗尽内存。`capacity` 和 `max_samples_per_send` 的默认值会将分片内存使用限制为每个分片小于 100 kB。

## 参数 <a id="parameters"></a>

所有位于远程写配置`queue_config`部分的相关参数

### `capacity`

`capacity`\(容量\)控制在阻止从 WAL 读取之前，每个分片在内存中排队多少个样本。一旦 WAL 被阻止，就无法将样本附加到任何分片，并且所有吞吐量都将停止。

在大多数情况下，容量应足够高以避免阻塞其他分片，但是太大的容量可能会导致过多的内存消耗，并导致重新分片期间清除队列的时间更长。建议将 `capacity` 参数设置为 `max_samples_per_send` 的 3-10 倍。

### `max_shards`

`max_shards`\(最大分片数\)配置最大分片数或并行性，Prometheus 将为每个远程写入队列使用。Prometheus 尝试不使用过多的分片，但是如果队列落后于远程写组件，则会将分片的数量增加到最大分片数以增加吞吐量。除非远程写入非常慢的端点，否则`max_shards`不能增加到默认值以上。如果有可能使远程端点不堪重负，则可能需要减少最大分片数，或者在备份数据时减少内存使用量。

### `min_shards`

`min_shards`\(最小分片数\)配置 Prometheus 使用的最小分片数量，并且是远程写入开始时使用的分片数量。如果远程写入落后，Prometheus 将自动扩大分片的数量，因此大多数用户不必调整此参数。增加最小分片数将使 Prometheus 在计算所需分片数时避免在一开始就落后。

### `max_samples_per_send`

`max_samples_per_send`\(每次发送的最大样本数\)可以根据使用的后端进行调整。许多系统可以通过每批发送更多样本而不会显着增加延迟来很好地工作。如果尝试在每个请求中发送大量样本，则其他后端将出现问题。默认值足够小，可以在大多数系统上使用。

### `batch_send_deadline`

`batch_send_deadline`\(批量发送超时时间\)设置单个分片发送的最大超时时间。即使排队的分片尚未达到`max_samples_per_send`，也会发送请求。对于对延迟不敏感的小批量系统，可以增加批量发送的截止时间，以提高请求效率。

### `min_backoff`

`min_backoff`\(失败的最小重试时间\)控制重试失败请求之前等待的最短时间。当远程端点重新连接时，增加重试时间将分散请求。对于每个失败的请求，重试时间间隔都会加倍，直到增加到`max_backoff`。

### `max_backoff`

`max_backoff`\(失败的最大重试时间\)控制重试失败请求之前等待的最长时间。

