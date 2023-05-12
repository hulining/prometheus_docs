# 配置

[Alertmanager](https://github.com/prometheus/alertmanager) 通过命令行标志和配置文件进行配置。命令行标志配置不可变的系统参数，配置文件定义抑制规则，通知路由和通知接收者。

[visual editor](https://prometheus.io/webtools/alerting/routing-tree-editor) 可以帮助构建路由树。

要查看所有可用的命令行标志，请运行`alertmanager -h`.

Alertmanager 可以在运行时重新加载配置。如果新配置格式不正确，则不会应用更改，且错误会被输出。通过发送`SIGHUP`信号到 Alertmanager 进程或发送 HTTP POST 请求到`/-/reload`端点触发配置重新加载。

## 配置文件 <a href="#configuration-file" id="configuration-file"></a>

要指定要加载的配置文件，请使用`--config.file`标志。

```bash
./alertmanager --config.file=alertmanager.yml
```

该文件是 [YAML 格式](https://en.wikipedia.org/wiki/YAML) 的，由以下所述格式进行定义。方括号表示参数是可选的。对于非列表参数，该值设置为指定的默认值。

通用占位符定义如下:

* `<duration>`: 可被正则表达式`[0-9]+(ms|[smhdwy]`匹配的一段时间
* `<labelname>`: 可被正则表达式`[a-zA-Z_][a-zA-Z0-9_]*`匹配的字符串
* `<labelvalue>`: unicode 字符串
* `<filename>`: 当前工作目录中的合法的路径
* `<boolean>`: 值为 `true` 或 `false` 的布尔值
* `<string>`: 常规字符串
* `<secret>`: 加密后的常规字符串，例如密码
* `<tmpl_string>`: 使用前已被模版扩展的字符串
* `<tmpl_secret>`: 使用前已被模版扩展的加密字符串

其他占位符分别指定。

提供的合法示例文件显示了上下文中的用法。

全局配置指定在所有其他配置上下文中有效的参数。它们还用作其他配置部分的默认设置。

```yaml
global:
  # 默认的 SMTP From 头字段
  [ smtp_from: <tmpl_string> ]
  # 用于发送邮件的默认 SMTP 主机，包括端口号。
  # 端口号通常为25，对于支持 TLS 的 SMTP(有时称为STARTTLS)为587
  # 示例: smtp.example.org:587
  [ smtp_smarthost: <string> ]
  # 标识到 SMTP 服务器的默认主机名
  [ smtp_hello: <string> | default = "localhost" ]
  # SMTP 身份认证相关
  # SMTP Auth using CRAM-MD5, LOGIN and PLAIN. If empty, Alertmanager doesn't authenticate to the SMTP server.
  [ smtp_auth_username: <string> ]
  # SMTP Auth using LOGIN and PLAIN.
  [ smtp_auth_password: <secret> ]
  # SMTP Auth using PLAIN.
  [ smtp_auth_identity: <string> ]
  # SMTP Auth using CRAM-MD5. 
  [ smtp_auth_secret: <secret> ]
  # SMTP 默认使用 TLS
  # 请注意，Go不支持与远程 SMTP 端点的未加密连接
  [ smtp_require_tls: <bool> | default = true ]

  # 用于 Slack 通知的API URL
  [ slack_api_url: <secret> ]
  [ victorops_api_key: <secret> ]
  [ victorops_api_url: <string> | default = "https://alert.victorops.com/integrations/generic/20131114/alert/" ]
  [ pagerduty_url: <string> | default = "https://events.pagerduty.com/v2/enqueue" ]
  [ opsgenie_api_key: <secret> ]
  [ opsgenie_api_url: <string> | default = "https://api.opsgenie.com/" ]
  [ hipchat_api_url: <string> | default = "https://api.hipchat.com/" ]
  [ hipchat_auth_token: <secret> ]
  [ wechat_api_url: <string> | default = "https://qyapi.weixin.qq.com/cgi-bin/" ]
  [ wechat_api_secret: <secret> ]
  [ wechat_api_corp_id: <string> ]

  # 默认的 HTTP 配置
  [ http_config: <http_config> ]

  # 如果警报不设置 EndsAt，则 ResolveTimeout 是 Alertmanager 使用的默认值，经过此时间后，如果告警尚未更新，则可以将告警声明为已解决
  # 这对 Prometheus 的告警没有任何影响，因为它们始终包含 EndsAt
  [ resolve_timeout: <duration> | default = 5m ]

# 从中读取自定义通知模板定义的文件.
# 最后一个组件可以使用通配符匹配器,如 'templates/*.tmpl'.
templates:
  [ - <filepath> ... ]

# 路由树的根节点
route: <route>

# 通知接收者列表.
receivers:
  - <receiver> ...

# 告警抑制规则列表.
inhibit_rules:
  [ - <inhibit_rule> ... ]
```

## `<route>` <a href="#route" id="route"></a>

路由配置块定义了路由树种的节点及其子节点。如果未设置，则其可选配置参数将从父节点继承。

每个警报都会在已配置的顶级路由处进入路由树，该路由树必须与所有警报匹配(即没有任何已配置的匹配器)。然后，它遍历子节点。如果`continue`设置为 false，它将在第一个匹配的子项之后停止。如果`continue`设置为 true，则告警将继续与后续的同级进行匹配。如果告警与节点的任何子节点都不匹配(不匹配的子节点或不存在子节点)，则根据当前节点的配置参数来处理告警。

```yaml
[ receiver: <string> ]
# 用于将传入警报分组在一起的标签. 例如, 对于 cluster = A 和 alertname = LatencyHigh 的多个警报将被分为一个组
#
# 要按所有可能的标签进行汇总,请使用特殊值 '...' 作为唯一的标签名称.例如: 
# group_by：['...']
# 这样可以有效地完全禁用聚合,并按原样传递所有警报.除非您的警报量非常低或上游通知系统执行自己的分组规则,否则这不太可能是您想要的
[ group_by: '[' <labelname>, ... ']' ]

# 告警是否应继续匹配后续的同级节点
[ continue: <boolean> | default = false ]

# 告警必须满足等值匹配才能匹配节点
match:
  [ <labelname>: <labelvalue>, ... ]

# 警必须满足正则匹配才能匹配节点
match_re:
  [ <labelname>: <regex>, ... ]

# 为一组警报发送通知的最初等待时间.等待抑制告警到达或为同一组收集更多初始警报
[ group_wait: <duration> | default = 30s ]

# 发送有关新告警的通知之前要等待的时间.该通知将添加到已为其发送了初始通知的一组警报中
[ group_interval: <duration> | default = 5m ]

# 如果已成功发送告警通知,等待多长时间才能再次发送通知.
[ repeat_interval: <duration> | default = 4h ]

# 0 个或多个子路由.
routes:
  [ - <route> ... ]
```

### 示例 <a href="#example" id="example"></a>

```yaml
# 具有所有参数的根路由.如果未覆盖,则由子路由继承
route:
  receiver: 'default-receiver'
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  group_by: [cluster, alertname]
  # 与以下子路由不匹配的所有告警将保留在根节点上,并分派给 'default-receiver'.
  routes:
  # 所有带有 service = mysql或 service = cassandra 的告警都将分派到 'database-pager'
  - receiver: 'database-pager'
    group_wait: 10s
    match_re:
      service: mysql|cassandra
  # 所有带有 team = frontend 标签的警报均与此子路由匹配.它们按 product 和 environment 分组,而不是按群集和警报名称分组
  - receiver: 'frontend-pager'
    group_by: [product, environment]
    match:
      team: frontend
```

## `<inhibit_rule>` <a href="#inhibit_rule" id="inhibit_rule"></a>

当存在与另一组匹配器匹配的告警(source)时，抑制规则会使与一组匹配器匹配的告警(target)静音。目标告警和源告警在`equal`列表中的标签名称都必须具有相同的标签值。

从语义上说，缺少标签和带有空值的标签是同一件事。因此，如果源警报和目标警报都缺少所有`equal`列出的标签名称，则应用抑制规则。

为了防止告警抑制自身，抑制规则将永远不会抑制与规则的目标和源侧匹配的告警。但是，我们建议选择目标匹配器和源匹配器，以使警报永远不会匹配双方。这更加方便，并且不会触发这种特殊情况。

```yaml
# 必须在警报中满足的匹配器才能静音
target_match:
  [ <labelname>: <labelvalue>, ... ]
target_match_re:
  [ <labelname>: <regex>, ... ]

# 必须存在一个或多个告警能使抑制生效的匹配器
source_match:
  [ <labelname>: <labelvalue>, ... ]
source_match_re:
  [ <labelname>: <regex>, ... ]

# 必须在源告警和目标告警中具有相等值的标签才能使抑制生效
[ equal: '[' <labelname>, ... ']' ]
```

## `<http_config>` <a href="#http_config" id="http_config"></a>

`http_config`允许配置接收者用来与基于 HTTP API 服务进行通信的 HTTP 客户端。

```yaml
# 注意 `basic_auth`, `bearer_token` 和 `bearer_token_file` 选项是互斥的

# 使用配置的用户名和密码设置 `Authorization` 请求头
# password 和 password_file 是互斥的
basic_auth:
  [ username: <string> ]
  [ password: <secret> ]
  [ password_file: <string> ]

# 使用配置的承载令牌设置 `Authorization` 请求头
[ bearer_token: <secret> ]

# 使用配置的承载令牌文件设置 `Authorization` 请求头
[ bearer_token_file: <filepath> ]

# TLS 配置
tls_config:
  [ <tls_config> ]

# 可选的代理 URL.
[ proxy_url: <string> ]
```

## `<tls_config>` <a href="#tls_config" id="tls_config"></a>

`tls_config`允许配置 TLS 连接

```yaml
# 用于验证服务器证书的 CA 证书
[ ca_file: <filename> ]

# 用于服务器的客户端证书认证的证书和密钥文件
[ cert_file: <filename> ]
[ key_file: <filename> ]

# ServerName 指定服务器的名称
# https://tools.ietf.org/html/rfc4366#section-3.1
[ server_name: <string> ]

# 禁用服务器证书的验证
[ insecure_skip_verify: <boolean> ]
```

## `<receiver>` <a href="#receiver" id="receiver"></a>

`receiver`是一个或多个通知集成的命名配置。

**我们不会积极添加新的接收器，我们建议通过** [**webhook**](configuration.md#webhook\_config) **receiver 实现自定义通知集成。**

```yaml
# receiver 的唯一名称
name: <string>

# 多个通知集成的配置
email_configs:
  [ - <email_config>, ... ]
hipchat_configs:
  [ - <hipchat_config>, ... ]
pagerduty_configs:
  [ - <pagerduty_config>, ... ]
pushover_configs:
  [ - <pushover_config>, ... ]
slack_configs:
  [ - <slack_config>, ... ]
opsgenie_configs:
  [ - <opsgenie_config>, ... ]
webhook_configs:
  [ - <webhook_config>, ... ]
victorops_configs:
  [ - <victorops_config>, ... ]
wechat_configs:
  [ - <wechat_config>, ... ]
```

## `<email_config>` <a href="#email_config" id="email_config"></a>

```yaml
# 是否通知告警已解决.
[ send_resolved: <boolean> | default = false ]

# 发送通知的电子邮件地址.
to: <tmpl_string>

# 发件人地址.
[ from: <tmpl_string> | default = global.smtp_from ]

# 发送邮件的 SMTP 主机.
[ smarthost: <string> | default = global.smtp_smarthost ]

# SMTP 服务器的主机名标识.
[ hello: <string> | default = global.smtp_hello ]

# SMTP 认证信息.
[ auth_username: <string> | default = global.smtp_auth_username ]
[ auth_password: <secret> | default = global.smtp_auth_password ]
[ auth_secret: <secret> | default = global.smtp_auth_secret ]
[ auth_identity: <string> | default = global.smtp_auth_identity ]

# SMTP TLS要求.
# 请注意，Go不支持与远程 SMTP 端点的未加密连接
[ require_tls: <bool> | default = global.smtp_require_tls ]

# TLS 配置.
tls_config:
  [ <tls_config> ]

# 邮件通知的HTML正文.
[ html: <tmpl_string> | default = '{{ template "email.default.html" . }}' ]
# 邮件通知的文本正文.
[ text: <tmpl_string> ]

# 电子邮件头键/值对。 覆盖先前由通知实现设置的所有请求头
[ headers: { <string>: <tmpl_string>, ... } ]
```

## `<hipchat_config>` <a href="#hipchat_config" id="hipchat_config"></a>

HipChat 通知使用 [Build Your Own](https://confluence.atlassian.com/hc/integrations-with-hipchat-server-683508267.html) 集成.

```yaml
# 是否通知告警已解决.
[ send_resolved: <boolean> | default = false ]

# The HipChat Room ID.
room_id: <tmpl_string>
# The auth token.
[ auth_token: <secret> | default = global.hipchat_auth_token ]
# The URL to send API requests to.
[ api_url: <string> | default = global.hipchat_api_url ]

# See https://www.hipchat.com/docs/apiv2/method/send_room_notification
# 除了发送人名称之外,还将显示的标签
[ from:  <tmpl_string> | default = '{{ template "hipchat.default.from" . }}' ]
# 消息体.
[ message:  <tmpl_string> | default = '{{ template "hipchat.default.message" . }}' ]
# 此消息是否应触发用户通知.
[ notify:  <boolean> | default = false ]
# 确定该消息如何被 alertmanager 处理并在 HipChat 中呈现.可选值是 'text' 和 'html'.
[ message_format:  <string> | default = 'text' ]
# 消息的背景色.
[ color:  <tmpl_string> | default = '{{ if eq .Status "firing" }}red{{ else }}green{{ end }}' ]

# HTTP 客户端的配置
[ http_config: <http_config> | default = global.http_config ]
```

## `<pagerduty_config>` <a href="#pagerduty_config" id="pagerduty_config"></a>

PagerDuty 通知是通过 [PagerDuty API](https://developer.pagerduty.com/documentation/integration/events) 发送的。PagerDuty 提供了有关如何集成的[文档](https://www.pagerduty.com/docs/guides/prometheus-integration-guide/)。Alertmanager v0.11 之后的版本重要的区别在于对PagerDuty的 Events API v2 的支持。

```yaml
# 是否通知告警已解决.
[ send_resolved: <boolean> | default = true ]

# 以下两个选项是互斥的
# The PagerDuty integration key (when using PagerDuty integration type `Events API v2`).
routing_key: <tmpl_secret>
# The PagerDuty integration key (when using PagerDuty integration type `Prometheus`).
service_key: <tmpl_secret>

# The URL to send API requests to
[ url: <string> | default = global.pagerduty_url ]

# Alertmanager 的客户端标识
[ client:  <tmpl_string> | default = '{{ template "pagerduty.default.client" . }}' ]
# 到通知发送者的反向链接.
[ client_url:  <tmpl_string> | default = '{{ template "pagerduty.default.clientURL" . }}' ]

# 事件的描述.
[ description: <tmpl_string> | default = '{{ template "pagerduty.default.description" .}}' ]

# 通知时间的严重程度.
[ severity: <tmpl_string> | default = 'error' ]

# 一组任意键/值对,可提供有关事件的更多详细信息
[ details: { <string>: <tmpl_string>, ... } | default = {
  firing:       '{{ template "pagerduty.default.instances" .Alerts.Firing }}'
  resolved:     '{{ template "pagerduty.default.instances" .Alerts.Resolved }}'
  num_firing:   '{{ .Alerts.Firing | len }}'
  num_resolved: '{{ .Alerts.Resolved | len }}'
} ]

# 附加到通知的图片.
images:
  [ <image_config> ... ]

# 附加到通知的链接.
links:
  [ <link_config> ... ]

# HTTP 客户端配置.
[ http_config: <http_config> | default = global.http_config ]
```

### `<image_config>` <a href="#image_config" id="image_config"></a>

这些字段记录在 [PagerDuty API 文档](https://v2.developer.pagerduty.com/v2/docs/send-an-event-events-api-v2#section-the-images-property)中

```yaml
href: <tmpl_string>
source: <tmpl_string>
alt: <tmpl_string>
```

### `<link_config>` <a href="#link_config" id="link_config"></a>

这些字段记录在 [PagerDuty API 文档](https://v2.developer.pagerduty.com/v2/docs/send-an-event-events-api-v2#section-the-images-property)中

```yaml
href: <tmpl_string>
text: <tmpl_string>
```

## `<pushover_config>` <a href="#pushover_config" id="pushover_config"></a>

Pushover 通知是通过 [Pushover API](https://pushover.net/api) 发送的。

```yaml
# 是否通知告警已解决.
[ send_resolved: <boolean> | default = true ]

# 收件人的用户密钥
user_key: <secret>

# 您注册的应用程序的 API 令牌, 参阅 https://pushover.net/apps
token: <secret>

# 通知标题.
[ title: <tmpl_string> | default = '{{ template "pushover.default.title" . }}' ]

# 通知消息.
[ message: <tmpl_string> | default = '{{ template "pushover.default.message" . }}' ]

# 消息旁边显示的补充 URL
[ url: <tmpl_string> | default = '{{ template "pushover.default.url" . }}' ]

# 优先级, 参阅 https://pushover.net/api#priority
[ priority: <tmpl_string> | default = '{{ if eq .Status "firing" }}2{{ else }}0{{ end }}' ]

# Pushover 服务器多久发送一次相同的通知给用户
# 必须至少 30 秒
[ retry: <duration> | default = 1m ]

# 除非用户确认通知,否则将继续重试通知的时间
[ expire: <duration> | default = 1h ]

# HTTP 客户端配置.
[ http_config: <http_config> | default = global.http_config ]
```

## `<slack_config>` <a href="#slack_config" id="slack_config"></a>

Slack 通知是通过 [Slack Webhooks](https://api.slack.com/incoming-webhooks) 发送的。通知包含[附件](https://api.slack.com/docs/message-attachments)。

```yaml
# 是否通知告警已解决.
[ send_resolved: <boolean> | default = false ]

# The Slack webhook URL.
[ api_url: <secret> | default = global.slack_api_url ]

# 发送通知的频道或用户
channel: <tmpl_string>

# Slack webhook API 定义的 API 请求数据
[ icon_emoji: <tmpl_string> ]
[ icon_url: <tmpl_string> ]
[ link_names: <boolean> | default = false ]
[ username: <tmpl_string> | default = '{{ template "slack.default.username" . }}' ]
# 以下参数定义附件.
actions:
  [ <action_config> ... ]
[ callback_id: <tmpl_string> | default = '{{ template "slack.default.callbackid" . }}' ]
[ color: <tmpl_string> | default = '{{ if eq .Status "firing" }}danger{{ else }}good{{ end }}' ]
[ fallback: <tmpl_string> | default = '{{ template "slack.default.fallback" . }}' ]
fields:
  [ <field_config> ... ]
[ footer: <tmpl_string> | default = '{{ template "slack.default.footer" . }}' ]
[ mrkdwn_in: '[' <string>, ... ']' | default = ["fallback", "pretext", "text"] ]
[ pretext: <tmpl_string> | default = '{{ template "slack.default.pretext" . }}' ]
[ short_fields: <boolean> | default = false ]
[ text: <tmpl_string> | default = '{{ template "slack.default.text" . }}' ]
[ title: <tmpl_string> | default = '{{ template "slack.default.title" . }}' ]
[ title_link: <tmpl_string> | default = '{{ template "slack.default.titlelink" . }}' ]
[ image_url: <tmpl_string> ]
[ thumb_url: <tmpl_string> ]

# HTTP 客户端配置.
[ http_config: <http_config> | default = global.http_config ]
```

### `<action_config>` <a href="#action_config" id="action_config"></a>

这些字段记录在 Slack API 文档中，用于[附件消息](https://api.slack.com/docs/message-attachments#action\_fields)和[交互式消息](https://api.slack.com/docs/interactive-message-field-guide#action\_fields).

```yaml
text: <tmpl_string>
type: <tmpl_string>
# url 或名称和值都是必需的
[ url: <tmpl_string> ]
[ name: <tmpl_string> ]
[ value: <tmpl_string> ]

[ confirm: <action_confirm_field_config> ]
[ style: <tmpl_string> | default = '' ]
```

#### `<action_confirm_field_config>` <a href="#action_confirm_field_config" id="action_confirm_field_config"></a>

这些字段记录在 [Slack API 文档](https://api.slack.com/docs/interactive-message-field-guide#confirmation\_fields)中。

```yaml
[ dismiss_text: <tmpl_string> | default '' ]
[ ok_text: <tmpl_string> | default '' ]
[ title: <tmpl_string> | default '' ]
text: <tmpl_string>
```

### `<field_config>` <a href="#field_config" id="field_config"></a>

这些字段记录在 [Slack API 文档](https://api.slack.com/docs/message-attachments#fields)中。

```yaml
title: <tmpl_string>
value: <tmpl_string>
[ short: <boolean> | default = slack_config.short_fields ]
```

## `<opsgenie_config>` <a href="#opsgenie_config" id="opsgenie_config"></a>

OpsGenie 通知是通过 [OpsGenie API](https://docs.opsgenie.com/docs/alert-api) 发送的。

```yaml
# 是否通知告警已解决.
[ send_resolved: <boolean> | default = true ]

# 与 OpsGenie API 通信时使用的 API 密钥
[ api_key: <secret> | default = global.opsgenie_api_key ]

# 将 OpsGenie API 请求发送到
[ api_url: <string> | default = global.opsgenie_api_url ]

# 告警文本不得超过130个字符.
[ message: <tmpl_string> ]

# 事件描述.
[ description: <tmpl_string> | default = '{{ template "opsgenie.default.description" . }}' ]

# 到通知发送者的反向链接.
[ source: <tmpl_string> | default = '{{ template "opsgenie.default.source" . }}' ]

# 一组任意键/值对,它们提供有关事件的更多详细信息
[ details: { <string>: <tmpl_string>, ... } ]

# 负责通知的响应者列表
responders:
  [ - <responder> ... ]

# 逗号分隔的附加到通知的标记列表
[ tags: <tmpl_string> ]

# 附加警报提示
[ note: <tmpl_string> ]

# 告警优先级,可能的值是 P1, P2, P3, P4, P5
[ priority: <tmpl_string> ]

# HTTP 客户端配置.
[ http_config: <http_config> | default = global.http_config ]
```

### `<responder>` <a href="#responder" id="responder"></a>

```yaml
# 应该定义这些字段之一
[ id: <tmpl_string> ]
[ name: <tmpl_string> ]
[ username: <tmpl_string> ]

# "team", "user", "escalation" 或 schedule".
type: <tmpl_string>
```

## `<victorops_config>` <a href="#victorops_config" id="victorops_config"></a>

VictorOps 通知通过 [VictorOps API](https://help.victorops.com/knowledge-base/victorops-restendpoint-integration/) 发送。

```yaml
# 是否通知告警已解决.
[ send_resolved: <boolean> | default = true ]

# 与 VictorOps API 通信时使用的 API 密钥
[ api_key: <secret> | default = global.victorops_api_key ]

# The VictorOps API URL.
[ api_url: <string> | default = global.victorops_api_url ]

#  A key used to map the alert to a team.
routing_key: <tmpl_string>

# 描述告警级别(CRITICAL, WARNING, INFO).
[ message_type: <tmpl_string> | default = 'CRITICAL' ]

# 包含警报问题的摘要
[ entity_display_name: <tmpl_string> | default = '{{ template "victorops.default.entity_display_name" . }}' ]

# 包含有关告警问题的详细说明
[ state_message: <tmpl_string> | default = '{{ template "victorops.default.state_message" . }}' ]

# 来自监控工具的状态消息
[ monitoring_tool: <tmpl_string> | default = '{{ template "victorops.default.monitoring_tool" . }}' ]

# HTTP 客户端配置.
[ http_config: <http_config> | default = global.http_config ]
```

## `<webhook_config>` <a href="#webhook_config" id="webhook_config"></a>

Webhook receiver 允许配置通用 receiver.

```yaml
# 是否通知告警已解决.
[ send_resolved: <boolean> | default = true ]

# 发送 HTTP POST 请求的端点
url: <string>

# HTTP 客户端配置.
[ http_config: <http_config> | default = global.http_config ]
```

Alertmanager 将以以下 JSON 格式将 HTTP POST 请求发送到配置的端点

```
{
  "version": "4",
  "groupKey": <string>,    // key identifying the group of alerts (e.g. to deduplicate)
  "status": "<resolved|firing>",
  "receiver": <string>,
  "groupLabels": <object>,
  "commonLabels": <object>,
  "commonAnnotations": <object>,
  "externalURL": <string>,  // backlink to the Alertmanager.
  "alerts": [
    {
      "status": "<resolved|firing>",
      "labels": <object>,
      "annotations": <object>,
      "startsAt": "<rfc3339>",
      "endsAt": "<rfc3339>",
      "generatorURL": <string> // identifies the entity that caused the alert
    },
    ...
  ]
}
```

此功能的[集成列表](../operating/integrations.md#alertmanager-webhook-receiver)

## `<wechat_config>` <a href="#wechat_config" id="wechat_config"></a>

微信通知通过[微信 API](https://admin.wechat.com/wiki/index.php?title=Customer\_Service\_Messages) 发送。

```yaml
# 是否通知告警已解决.
[ send_resolved: <boolean> | default = false ]

# 与微信 API 通信时使用的 API 密钥
[ api_secret: <secret> | default = global.wechat_api_secret ]

# The WeChat API URL.
[ api_url: <string> | default = global.wechat_api_url ]

# 用于身份验证的公司 ID
[ corp_id: <string> | default = global.wechat_api_corp_id ]

# 由微信 API 定义的 API 请求数据
[ message: <tmpl_string> | default = '{{ template "wechat.default.message" . }}' ]
[ agent_id: <string> | default = '{{ template "wechat.default.agent_id" . }}' ]
[ to_user: <string> | default = '{{ template "wechat.default.to_user" . }}' ]
[ to_party: <string> | default = '{{ template "wechat.default.to_party" . }}' ]
[ to_tag: <string> | default = '{{ template "wechat.default.to_tag" . }}' ]
```
