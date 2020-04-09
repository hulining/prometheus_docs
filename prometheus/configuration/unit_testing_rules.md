---
title: 规则的单元测试
---

# 规则的单元测试

您可以使用`promtool`测试您的规则。

```bash
# 单独测试一个文件
./promtool test rules test.yml

# 如果你有多个测试文件, test1.yml,test2.yml,test2.yml
./promtool test rules test1.yml test2.yml test3.yml
```

## 测试文件格式

```yaml
# 要进行测试的规则文件的列表。支持globs。
rule_files:
  [ - <file_name> ]

# 可选的， default = 1m
evaluation_interval: <duration>

# 下面列出的组名的顺序僵尸规则组的执行顺序(在给定的执行时间内)。仅针对以下提及的组保证顺序。不需要体积所有组
group_eval_order:
  [ - <group_name> ]

# 所有测试均在此处列出
tests:
  [ - <test_group> ]
```

### `<test_group>`

```yaml
# 数据序列
interval: <duration>
input_series:
  [ - <series> ]

# 对上述数据进行单元测试

# 告警规则的单元测试。我们考虑输入文件中的告警规则
alert_rule_test:
  [ - <alert_test_case> ]

# PromQL 表达式的单元测试
promql_expr_test:
  [ - <promql_test_case> ]

# 告警模版可访问的外部扩展标签
external_labels:
  [ <labelname>: <string> ... ]
```

### `<series>`

```yaml
# 遵循常规的序列符号 '<metric name>{<label name>=<label value>, ...}'
# Examples:
#      series_name{label1="value1", label2="value2"}
#      go_goroutines{job="prometheus", instance="localhost:9090"}
series: <string>

# 使用扩展符号
# Expanding notation:
#     'a+bxc' becomes 'a a+b a+(2*b) a+(3*b) … a+(c*b)'
#     'a-bxc' becomes 'a a-b a-(2*b) a-(3*b) … a-(c*b)'
# Examples:
#     1. '-2+4x3' becomes '-2 2 6 10'
#     2. ' 1-2x4' becomes '1 -1 -3 -5 -7'
values: <string>
```

### `<alert_test_case>`

Prometheus 允许您为不通的告警规则使用相同的告警名称。因此，在此单元测试中，您必须在单个`<alert_test_case>`下列出警报名称的所有触发告警的并集。

```yaml
# 检查告警的时间间隔
eval_time: <duration>

# 要测试的告警名称
alertname: <string>

# 给定执行时间以给定警报名称触发的预期告警列表。 如果要测试是否不应该触发警报规则，则可以提及上述字段，并将 "exp_alerts" 留空。
exp_alerts:
 [ - <alert> ]
```

### `<alert>`

```yaml
# 预期告警的扩展标签和注解.
# 注意: 标签包括与告警关联的样本的标签(与您在 `/alerts` 中看到的标签相同，但不包含 `__name__` 和 `alertname` 序列)
exp_labels:
  [ <labelname>: <string> ]
exp_annotations:
  [ <labelname>: <string> ]
```

```yaml
# 执行的表达式
expr: <string>

# 执行表达式的时间间隔
eval_time: <duration>

# 给定评估时间的预期样本
exp_samples:
  [ - <sample> ]
```

### `<sample>`

```yaml
# 数据样本的标签采用常规系列标记 '<metric name>{<label name>=<label value>, ...}'
# Examples:
#      series_name{label1="value1", label2="value2"}
#      go_goroutines{job="prometheus", instance="localhost:9090"}
labels: <string>

# PromQL 表达式的期望值.
value: <number>
```

## 示例

这是通过测试的单元测试的示例输入文件。`test.yml`是遵循上述语法的测试文件，`alert.yml`包含警报规则。

将`alerts.yml`放在同一目录中，运行`./promtool test rules test.yml`.

### `test.yml`

```yaml
# 这是单元测试的入口
# 仅将此文件作为命令行参数传递

rule_files:
    - alerts.yml

evaluation_interval: 1m

tests:
    # Test 1.
    - interval: 1m
      # Series data.
      input_series:
          - series: 'up{job="prometheus", instance="localhost:9090"}'
            values: '0 0 0 0 0 0 0 0 0 0 0 0 0 0 0'
          - series: 'up{job="node_exporter", instance="localhost:9100"}'
            values: '1+0x6 0 0 0 0 0 0 0 0' # 1 1 1 1 1 1 1 0 0 0 0 0 0 0 0
          - series: 'go_goroutines{job="prometheus", instance="localhost:9090"}'
            values: '10+10x2 30+20x5' # 10 20 30 30 50 70 90 110 130
          - series: 'go_goroutines{job="node_exporter", instance="localhost:9100"}'
            values: '10+10x7 10+30x4' # 10 20 30 40 50 60 70 80 10 40 70 100 130

      # Unit test for alerting rules.
      alert_rule_test:
          # Unit test 1.
          - eval_time: 10m
            alertname: InstanceDown
            exp_alerts:
                # Alert 1.
                - exp_labels:
                      severity: page
                      instance: localhost:9090
                      job: prometheus
                  exp_annotations:
                      summary: "Instance localhost:9090 down"
                      description: "localhost:9090 of job prometheus has been down for more than 5 minutes."
      # Unit tests for promql expressions.
      promql_expr_test:
          # Unit test 1.
          - expr: go_goroutines > 5
            eval_time: 4m
            exp_samples:
                # Sample 1.
                - labels: 'go_goroutines{job="prometheus",instance="localhost:9090"}'
                  value: 50
                # Sample 2.
                - labels: 'go_goroutines{job="node_exporter",instance="localhost:9100"}'
                  value: 50
```

### `alert.yml`

```yaml
# This is the rules file.

groups:
- name: example
  rules:

  - alert: InstanceDown
    expr: up == 0
    for: 5m
    labels:
        severity: page
    annotations:
        summary: "Instance {{ $labels.instance }} down"
        description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."

  - alert: AnotherInstanceDown
    expr: up == 0
    for: 10m
    labels:
        severity: page
    annotations:
        summary: "Instance {{ $labels.instance }} down"
        description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."
```

