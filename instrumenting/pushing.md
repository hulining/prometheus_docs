---
title: 推送数据指标
---

# 推送数据指标

有时，您需要监控无法采集数据组件。[ Prometheus Pushgateway](https://github.com/prometheus/pushgateway) 允许您将时间序列从[短暂的服务级别批处理作业](../practices/pushing.md)推送到 Prometheus 可以采集的中间作业。结合 Prometheus 的基于文本的简单展示格式，即使没有客户端库，也可以轻松集成，甚至编写 shell 脚本。

* 有关使用 Pushgateway 及在 Unix Shell 中使用的更多信息，请参见该项目的 [README.md](https://github.com/prometheus/pushgateway/blob/master/README.md)。
* 要在 Java 中使用，请参见 [PushGateway](https://prometheus.github.io/client_java/io/prometheus/client/exporter/PushGateway.html)类。
* 要在 Go 中使用，请参见 [Push](https://godoc.org/github.com/prometheus/client_golang/prometheus/push#Pusher.Push) 和 [Add](https://godoc.org/github.com/prometheus/client_golang/prometheus/push#Pusher.Add) 方法。
* 要在 Python 使用，请参阅[导出到 Pushgateway](https://github.com/prometheus/client_python#exporting-to-a-pushgateway)。
* 要在 Ruby 中使用，请参见 [Pushgateway 文档](https://github.com/prometheus/client_ruby#pushgateway)
  * 要了解[在 Prometheus 项目之外维护的客户端库](clientlibs.md)的 Pushgateway 支持，请参阅其各自的文档。

