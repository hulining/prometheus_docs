# 通知模板示例

以下是告警和相应的 Alertmanager 配置文件设置(alertmanager.yml)的所有不同示例。每个都使用 [Go 模板](http://golang.org/pkg/text/template/) 系统。

## 自定义 Slack 通知 <a href="#customizing-slack-notifications" id="customizing-slack-notifications"></a>

在此示例中，我们自定义了 Slack 通知，以向组织的 Wiki 发送有关如何处理已发送的特定告警的 URL。

```
global:
  slack_api_url: '<slack_webhook_url>'

route:
  receiver: 'slack-notifications'
  group_by: [alertname, datacenter, app]

receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#alerts'
    text: 'https://internal.myorg.net/wiki/alerts/{{ .GroupLabels.app }}/{{ .GroupLabels.alertname }}'
```

## 访问 CommonAnnotations 中的注解 <a href="#accessing-annotations-in-commonannotations" id="accessing-annotations-in-commonannotations"></a>

在此示例中，我们以访问 Alertmanager 发送的数据的 `CommonAnnotations` 中存储的`summary`和`description`再次自定义发送到 Slack receiver 的通知 text。

Alert

```yaml
groups:
- name: Instances
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 5m
    labels:
      severity: page
    # Prometheus 模板在此处应用告警注释和标签字段
    annotations:
      description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes.'
      summary: 'Instance {{ $labels.instance }} down'
```

Receiver

```yaml
- name: 'team-x'
  slack_configs:
  - channel: '#alerts'
    # Alertmanager 模板在此处应用
    text: "<!channel> \nsummary: {{ .CommonAnnotations.summary }}\ndescription: {{ .CommonAnnotations.description }}"
```

## 覆盖所有收到的告警 <a href="#anging-over-all-received-alerts" id="anging-over-all-received-alerts"></a>

最后，假设告警与前面的示例相同，我们将接收器自定义为覆盖从 Alertmanager 收到的所有警报，并在新行上打印它们各自注解中的 summary 和 description。

Receiver

```
- name: 'default-receiver'
  slack_configs:
  - channel: '#alerts'
    title: "{{ range .Alerts }}{{ .Annotations.summary }}\n{{ end }}"
    text: "{{ range .Alerts }}{{ .Annotations.description }}\n{{ end }}"
```

## 定义可重用的模板 <a href="#defining-reusable-templates" id="defining-reusable-templates"></a>

回到第一个示例，我们还可以提供一个包含命名模板的文件，然后由 Alertmanager 加载该文件，以避免跨越多行的复杂模板。在`/alertmanager/template/myorg.tmpl`创建一个文件，并在其中创建一个名为`slack.myorg.txt`的模板:

```
{{ define "slack.myorg.text" }}https://internal.myorg.net/wiki/alerts/{{ .GroupLabels.app }}/{{ .GroupLabels.alertname }}{{ end}}
```

现在，如下配置将使用给定名称的 "text" 字段加载模板，并且我们提供了自定义模板文件的路径:

```
global:
  slack_api_url: '<slack_webhook_url>'

route:
  receiver: 'slack-notifications'
  group_by: [alertname, datacenter, app]

receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#alerts'
    text: '{{ template "slack.myorg.text" . }}'

templates:
- '/etc/alertmanager/templates/myorg.tmpl'
```

此[博客文章](https://prometheus.io/blog/2016/03/03/custom-alertmanager-templates/)中进一步详细说明了此示例。
