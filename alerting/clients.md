---
title: 发送告警
---

# 发送告警

{% hint style="warning" %}
**免责声明: Prometheus 自动处理发送由其配置的警报规则生成的警报。强烈建议根据时间序列数据在 Prometheus 中配置警报规则，而不要直接实现客户端。**
{% endhint %}

Alertmanager 有两个 API，v1 和 v2，都监听告警。v1 的方案在下面的代码中进行了描述。v2 的方案被指定为OpenAPI规范，可以在 [Alertmanager 代码仓库](https://github.com/prometheus/alertmanager/blob/master/api/v2/openapi.yaml)中找到该规范。只要客户端仍然处于活动状态\(通常在30秒到3分钟左右\)，它们就可以不断重新发送告警。客户可以通过 POST 请求将告警列表推送到 Alertmanager。

每个告警的标签用于标识告警的相同实例并执行重复数据删除。注解始终设置为最近收到的注释，并且不能标识告警。

`startsAt`和`endsAt`时间戳都是可选的。如果省略`startsAt`，则由 Alertmanager 分配为当前时间。仅在知道警报的结束时间时才设置`endsAt`。否则，它将被设置为自上次收到警报以来的时间。

`generatorURL`字段是唯一的反向链接，用于标识客户端中此告警的原因实例。

```text
[
  {
    "labels": {
      "alertname": "<requiredAlertName>",
      "<labelname>": "<labelvalue>",
      ...
    },
    "annotations": {
      "<labelname>": "<labelvalue>",
    },
    "startsAt": "<rfc3339>",
    "endsAt": "<rfc3339>",
    "generatorURL": "<generator_url>"
  },
  ...
]
```

