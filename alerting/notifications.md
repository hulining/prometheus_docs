---
title: 通知模板参考
---

# 通知模板参考

Prometheus 创建告警并将其发送到 Alertmanager，Alertmanager 随后根据标签将通知发送到不同的接收者。接收者可以是许多集成中的一种，包括: Slack, PagerDuty, 邮件或通过通用 Webhook 接口的自定义集成。

发送到接收方的通知是通过模板构造的。Alertmanager 带有默认模板，但也可以自定义。为避免混淆，必须注意 Alertmanager 模板与 [Prometheus 模板](../prometheus/configuration/template_reference.md)不同，但是 Prometheus 模板还包括告警规则标签/注解中的模板。

Alertmanager 的通知模板基于 [Go 模板](https://golang.org/pkg/text/template)系统。请注意，某些字段被认定为文本，而其他字段则被评估为 HTML，这会影响转义。

## 数据结构 <a id="data-structures"></a>

### Data <a id="data"></a>

`data`是传递给通知模板和 Webhook 推送的结构。

| 名称 | 类型 | 解释 |
| :---: | :---: | :---: |
| Receiver | string | 定义将通知发送到的接收者的名称\(slack, email等\) |
| Status | string | 如果至少一个警报正在触发，则定义为 firing，否则定义为 resolved |
| Alerts | [Alert](notifications.md#alert) | 该组中所有告警对象的列表\(参见下文\) |
| GroupLabels | [KV](notifications.md#kv) | 警报按指定标签分组 |
| CommonLabels | [KV](notifications.md#kv) | 所有告警共有的标签 |
| CommonAnnotations | [KV](notifications.md#kv) | 所有告警的通用注解集。用于获取有关警报的更多其他信息字符串 |
| ExternalURL | string | Alertmanager 发送通知的反向链接 |

`Alerts`类型公开了过滤告警的功能：

* `Alerts.Firing`返回此组中当前触发的告警对象的列表
* `Alerts.Resolved`返回此组中已解决的告警对象的列表

### Alert

`Alert`生成一个告警用于通知模板。

| 名称 | 类型 | 解释 |
| :---: | :---: | :---: |
| Status | string | 定义告警是 resolved 还是正在 firing |
| Labels | [KV](notifications.md#kv) | 一组附加到告警的标签 |
| Annotations | [KV](notifications.md#kv) | 一组附加到告警的注解 |
| StartsAt | time.Time | 告警开始触发的时间。如果省略，则由 Alertmanager 分配为当前时间 |
| EndsAt | time.Time | 仅在知道警报的结束时间时设置。否则，将其设置为自收到最后一个警报以来的时间 |
| GeneratorURL | string | 标识此告警原因的反向链接 |

### KV

`KV`是一组键/值字符串对，用于表示标签和注解

```text
type KV map[string]string
```

含有两个注解的注解示例:

```text
{
  summary: "alert summary",
  description: "alert description",
}
```

除了直接访问存储为 KV 的数据\(标签和注释\)外，还有一些用于排序，删除和查看 LabelSet 的方法

#### KV 方法 <a id="kv-methods"></a>

| 名称 | 参数 | 返回 | 解释 |
| :---: | :---: | :---: | :--- |
| SortedPairs | - | Pairs\(键/值字符串对的列表\) | 返回键/值对的排序列表 |
| Remove | \[\]string | KV | 返回没有给定键的键/值映射的副本 |
| Names | - | \[\]string | 返回 LabelSet 中所有标签名称 |
| Values | - | \[\]string | 返回 LabelSet 中所有标签值 |

## 函数 <a id="functions"></a>

Go 模板还提供了[默认函数](https://golang.org/pkg/text/template/#hdr-Functions)

| 名称 | 参数 | 返回 | 注解 |
| :---: | :---: | :---: | :---: |
| title | string | [strings.Title](http://golang.org/pkg/strings/#Title),每个单词的第一个字符大写 |  |
| toUpper | string | [strings.ToUpper](http://golang.org/pkg/strings/#ToUpper) | 将所有字符转换为大写. |
| toLower | string | [strings.ToLower](http://golang.org/pkg/strings/#ToLower) | 将所有字符转换为小写. |
| match | pattern, string | [Regexp.MatchString](https://golang.org/pkg/regexp/#MatchString) | 正则表达式匹配. |
| reReplaceAll | pattern, replacement, text | [Regexp.ReplaceAllString](http://golang.org/pkg/regexp/#Regexp.ReplaceAllString) 正则表达式替换, 未锚定. |  |
| join | sep string, s \[\]string | [strings.Join](http://golang.org/pkg/strings/#Join), 连接的元素以创建单个字符串。分隔符字符串sep放置在结果字符串中的元素之间. \(注意：参数顺序倒置以便在模板中更轻松地进行流水线.\) |  |
| safeHtml | text string | [html/template.HTML](https://golang.org/pkg/html/template/#HTML), 将字符串标记为HTML，不需要自动转义. |  |
| stringSlice | ...string | 以字符串切片的方式返回传递的字符串. |  |

