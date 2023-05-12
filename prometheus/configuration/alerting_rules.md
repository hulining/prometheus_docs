# 告警规则

告警规则允许您可以基于 Prometheus 表达式定义告警条件，并将有关发出告警的通知发送到到外部服务。只要告警表达式在给定的时间点生成一个或多个矢量元素，告警就被这些元素的标签集视为活动状态。

### 定义告警规则 <a href="#defining-alerting-rules" id="defining-alerting-rules"></a>

告警规则的配置方式与[记录规则](recording\_rules.md)相同。

示例告警规则文件如下：

```yaml
groups:
- name: example
  rules:
  - alert: HighRequestLatency
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
    for: 10m
    labels:
      severity: page
    annotations:
      summary: High request latency
```

可选的`for`子句使 Prometheus 在第一次遇到新的表达式输出向量元素与将告警即为对此元素触发进行计数之间等待一段时间。在本例中，Prometheus 在触发告警之前，检查10分钟内告警是否持续处于活动状态。处于活动状态但未触发的元素处于挂起状态。

`labels`子句指定一组标签添加到告警，与现有的标签冲突将会被覆盖。便签值可以模板化。

`annotations`子句指定一组信息标签，这些标签可用于存储更长的附加信息，例如告警描述或运行手册链接。注解值可以模板化。

### 模板化 <a href="#templating" id="templating"></a>

标签和注解值可以使用[控制台模板](../../visualization/consoles.md)模板化。`$labels`变量保存告警实例的标签键/值对。可以通过`$externalLabels`变量访问已配置的扩展标签。`$value`变量保存告警实例执行的值。

```yaml
# 输入告警实例的标签值
{{ $labels.<labelname> }}
# 输入触发告警元素的标志表达式的值
{{ $value }}
```

示例：

```yaml
groups:
- name: example
  rules:

  # Alert for any instance that is unreachable for >5 minutes.
  - alert: InstanceDown
    expr: up == 0
    for: 5m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."

  # Alert for any instance that has a median request latency >1s.
  - alert: APIHighRequestLatency
    expr: api_http_request_latencies_second{quantile="0.5"} > 1
    for: 10m
    annotations:
      summary: "High request latency on {{ $labels.instance }}"
      description: "{{ $labels.instance }} has a median request latency above 1s (current value: {{ $value }}s)"
```

### 在运行时检查告警 <a href="#inspecting-alerts-during-runtime" id="inspecting-alerts-during-runtime"></a>

要手动检查哪些告警处于活动状态(挂起或触发)，请导航到 Prometheus 实例的 "Alerts" 选项卡。该页面将向您展示每个定义的的告警当前处于活动状态的确切标签集。

对于挂起和触发的告警，Prometheus 存储格式为`ALERTS{alertname="<alert name>", alertstate="pending|firing", <additional alert labels>}`的时间序列。只要警报处于活动(挂起或触发)状态，样本值就设置为1；如果不再是这种情况，则将该系列标记为过时。

### 发送告警通知 <a href="#sending-alert-notifications" id="sending-alert-notifications"></a>

Prometheus 的告警规则擅长确定当前已经发生的问题，但它并不是完美的通知解决方案。在简单的告警定义之上，还需要另一层来添加摘要，通知速率限制，静默和告警依赖。在 Prometheus 生态系统中，[Alertmanager](../../alerting/alertmanager.md) 扮演了这个角色。因此，Prometheus 可以配置为定期将有关告警状态的信息发送到 Alertmanager 实例，该实例随后负责调度正确的通知。

可以将 Prometheus [配置](configuration.md)为通过其服务发现集成自动发现可用的 Alertmanager 实例。
