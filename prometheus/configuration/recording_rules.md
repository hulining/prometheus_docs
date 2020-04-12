---
title: 定义记录规则
---

# 定义记录规则

## 规则配置 <a id="configuring-rules"></a>

Prometheus 支持两种经过配置之后可以定期进行评估的规则：记录规则和[告警规则](alerting_rules.md)。要将规则应用在 Prometheus 中，请创建一个包含必要规则语句的文件，并通过 [Prometheus 配置](configuration.md)中的`rule_files`字段加载该文件。规则文件使用 YAML

通过发送`SIGHUP`信号到 Prometheus 进程，可以在运行时重新加载规则文件。仅当所有规则文件的格式正确时，才会应用更改。

## 规则语法检查 <a id="syntax-checking-rules"></a>

要在不启动 Prometheus 的情况下快速检查规则文件在语法上是否正确，请安装并运行 Prometheus 的`promtool`命令行工具：

```bash
go get github.com/prometheus/prometheus/cmd/promtool
promtool check rules /path/to/example.rules.yml
```

当该文件语法合法时，检查器将已解析的规则文本表达式打印到标准输出，然后退出，状态码为 0。

如果有任何语法错误或非法的输入参数，它将打印一条错误消息到标准错误，然后退出，状态码为 1。

## 记录规则 <a id="recording-rules"></a>

记录规则使您可以预先计算经常或计算量大的表达式，并将结果保存为一组新的时间序列。查询预计算结果通常比每次执行原始表达式要快得多。这对于每次刷新时都需要重复查询相同的表达式的仪表板特别有用。

记录和告警规则存在于规则组中，组中的规则以定期的时间间隔顺序运行。记录和告警规则的名称必须是[合法的数据指标名称](data_model.md#metric-names-and-labels)。

规则文件的语法是：

```yaml
groups:
  [ - <rule_group> ]
```

简单的示例规则文件如下：

```yaml
groups:
  - name: example
    rules:
    - record: job:http_inprogress_requests:sum
      expr: sum(http_inprogress_requests) by (job)
```

### `<rule_group>` <a id="rule_group"></a>

```yaml
# 组的名称。一个文件中必须唯一
name: <string>

# 评估组中规则的频率
[ interval: <duration> | default = global.evaluation_interval ]

rules:
  [ - <rule> ... ]
```

### `<rule>` <a id="rule"></a>

记录规则的语法如下：

```yaml
# 要输出到的新的时间序列的名称。必须是合法的数据指标名称
record: <string>

# 要执行的 PromQL 表达式。每隔执行周期的时间，表达式都会在当前时间进行执行，并将结果记录为一组新的时间序列，其数据指标名称由 "record" 给出
expr: <string>

# 存储结果之前要添加或覆盖的标签
labels:
  [ <labelname>: <labelvalue> ]
```

告警规则的语法如下：

```yaml
# 告警名称。必须是合法的数据指标名称
alert: <string>

# 要执行的 PromQL 表达式。每隔执行周期的时间，表达式都会在当前时间进行执行，所有结果时间序列都会变成  pending/firing 告警
expr: <string>

# 一旦持续返回告警多长时间，则将其视为 firing 告警
# 尚未触发足够长时间的告警被视为 pending
[ for: <duration> | default = 0s ]

# 为每个告警添加或覆盖的标签
labels:
  [ <labelname>: <tmpl_string> ]

# 为每个告警添加的注解
annotations:
  [ <labelname>: <tmpl_string> ]
```

