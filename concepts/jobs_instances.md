---
title: 作业和实例
---

# 作业和实例

用 Prometheus 的术语来说，可以进行采集数据指标的端点称为_实例_，通常对应于单独的进程。具有相同目地的实例的集合\(例如，可伸缩性或可靠性而复制的过程\)称为_作业_。

例如，具有四个复制实例的 API 作业：

* job: `api-server`
  * instance 1: `1.2.3.4.5670`
  * instance 2: `1.2.3.4.5671`
  * instance 3: `1.2.3.4.5672`
  * instance 4: `1.2.3.4.5673`

## 自动生成标签和时间序列 <a id="automatically-generated-labels-and-time-series"></a>

当 Prometheus 从目标采集数据指标时，它会自动在采集到的时间序列上附加一些标签，以便于识别被采集的目标：

* `job`: 采集数据目标所属的已配置的作业名称。
* `instance`: 采集数据目标 URL 的`<host>:<port>`部分。

如果这些标签中的任何一个已存在于采集的数据中，则行为取决于`honor_labels`配置选项。有关更多信息，请参见[采集配置文档](configuration.md#%3Cscrape_config%3E)。

对于每个实例的数据采集，Prometheus 按照以下时间序列存储样本：

* `up{job="<job-name>", instance="<instance-id>"}`: 如果实例运行状态良好，则为`1`；如果采集失败，则为`0`。
* `scrape_duration_seconds{job="<job-name>", instance="<instance-id>"}`: 采集的持续时间。
* `scrape_samples_post_metric_relabeling{job="<job-name>", instance="<instance-id>"}`: 数据指标重新标记后剩余样本数。
* `scrape_samples_scraped{job="<job-name>", instance="<instance-id>"}`: 目标暴露的样本数
* `scrape_series_added{job="<job-name>", instance="<instance-id>"}`: 本次采集新增样本数量。在 _v2.10_ 版本中新增

时间序列`up`对于实例可用性监控很有用。

