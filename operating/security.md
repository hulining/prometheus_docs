# 安全模型

Prometheus 是一个复杂的系统，它具有许多组件以及与其他系统的许多集成。它可以部署在各种受信任和不受信任的环境中。

本页介绍 Prometheus 的一般安全性假设以及某些配置可能存在的攻击媒介。

与任何复杂的系统一样，Prometheus 无法保证没有漏洞。如果发现安全漏洞，请私下向相关存储库的 MAINTAINERS.md 和 CC prometheus-team@googlegroups.com 中列出的维护人员报告。我们将解决问题并与您协调发布日期，确认您的努力的成果并且如果您想，我们可以提及您的名字。

## Prometheus

假定不受信任的用户可以访问 Prometheus HTTP 端点和日志。他们可以访问数据库中包含的所有时间序列信息，以及各种操作/调试信息。

还假定只有受信任的用户才能更改命令行，配置文件，规则文件以及 Prometheus 和其他组件的运行时环境等其他方面。

Prometheus 针对哪个目标采集数据，多久一次以及与其他任何设置完全通过配置文件确定。管理员可以决定使用来自服务发现的系统信息，将其与重新标记结合可以将某些控制权授予可以在该服务发现系统中修改数据的任何人。

对目标进行数据采集可能由不受信任的用户运行。默认情况下，目标不应公开体现其他目标的数据。`honor_labels`选项和某些重新标记设置移除了此保护机制。

从 Prometheus 2.0 开始，`--web.enable-admin-api`标志控制对管理型 HTTP API 的访问，其中包括删除时间序列等功能。默认情况下禁用。如果启用，则可以在`/api/*/admin/`路径下访问管理和更改功能。`--web.enable-lifecycle`标志控制 Prometheus 的 HTTP 重新加载和关闭。默认情况下也禁用此功能。如果启用，则可以在`/-/ reload`和`/-/ quit`路径下访问它们。

在Prometheus 1.x 中，有权访问 HTTP API 的任何人都可以访问`/-/reload`并在`/api/v1/series`上使用`DELETE`请求方法。默认情况下，`/-/quit`端点是禁用的，但可以通过`-web.enable-remote-shutdown`标志启用。

远程读取功能允许具有 HTTP 访问权限的任何人将查询发送到远程读取端点。例如，如果 PromQL 查询最终直接在关系数据库上运行，那么任何能够将查询发送到 Prometheus\(例如通过 Grafana\)的人都可以对该数据库运行任意 SQL。

## Alertmanager

有权访问 Alertmanager HTTP 端点的任何用户都可以访问其数据。他们可以创建和解决告警。他们可以创建，修改和删除静默内容。

通知发送到的位置由配置文件确定。使用某些模板设置，通知最终可以到达警报定义的目的地。例如，如果通知使用告警标签作为目标电子邮件地址，则可以将告警发送到 Alertmanager 的任何人都可以将通知发送到任何电子邮件地址。如果警报定义的目的地是可模板化的密文字段，则有权访问 Prometheus 或 Alertmanager 的任何人都可以查看该密文。

在以上用例中，任何可模板化的密文字段均用于路由通知。它们不旨在用作使用模板文件功能将密文与配置文件分离开的方法。能够在 Alertmanager 配置文件中配置接收者的任何人都可以泄露模板文件中存储的所有加密内容。例如，在大型架构中，每个团队都可能有一个他们完全控制的 alertmanager 配置文件片段，然后将其组合成完整的最终配置文件。

## Pushgateway

有权访问 Pushgateway HTTP 端点的任何用户都可以创建、修改和删除其中包含的数据指标。由于通常在启用了`honor_labels`的情况下采集 Pushgateway 数据，因此这意味着有权使用 Pushgateway 的任何人都可以在 Prometheus 中创建任何时间序列。

`--web.enable-admin-api`标志控制对管理型 HTTP API 的访问，其中包括清除所有现有数据指标组的功能。默认情况下禁用。如果启用，则可以在`/api/*/admin/`路径下访问管理功能。

## Exporters

Exporters 通常仅与具有一组预设的无法通过其 HTTP 端点进行扩展的命令/请求的配置实例进行通信。

还有一些 exporters ，例如 SNMP 和 Blackbox exporters，它们从 URL 参数获取目标。因此，对这些 exporter 具有 HTTP 访问权限的任何人都可以使它们将请求发送到任意目标端点。由于它们还支持客户端身份验证，因此可能导致诸如 HTTP 基本身份验证密码或 SNMP 认证之类的密钥泄露。请求响应身份验证机制\(例如 TLS\)不受此影响

## 客户端库 <a id="client-libraries"></a>

客户端库用于包含在用户的应用程序中。

如果使用客户端库提供的 HTTP 处理程序，则到达该处理程序的恶意请求应该不会引起由于负载过高和采集失败问题。

## 身份认证，授权和加密 <a id="authentication-authorization-and-encryption"></a>

Prometheus 及其组件不提供任何服务端身份验证，授权或加密功能。如果您需要这样做，建议使用反向代理。

由于打算通过简单的工具\(例如cURL\)访问管理和更改端点，因此没有内置的 [CSRF](https://en.wikipedia.org/wiki/Cross-site_request_forgery) 保护，因为这会破坏此类用例。因此，在使用反向代理时，您可能希望阻塞此类路径的请求以防止 CSRF 攻击。

对于非变化的端点，您可能想在反向代理中设置 [CORS 请求头](https://fetch.spec.whatwg.org/#http-cors-protocol)，例如`Access-Control-Allow-Origin`，以防止 [XSS](https://en.wikipedia.org/wiki/Cross-site_scripting) 攻击。

如果您要编写的 PromQL 查询包含不受信任的用户\(例如，控制台模板的 URL 参数或您自己创建的东西\)的输入，但这些输入不希望能够运行任意 PromQL 查询，请确保适当地转义了任何不受信任的输入以防止注入攻击。例如，如果`<user_input>`是`"} or some_metric{zzz="`，则`up{job="<user_input>"}`会变成`up{job=""} or some_metric{zzz=""}`

各种 Prometheus 组件均支持客户端身份验证和加密。如果提供 TLS 客户端支持，通常还会有一个跳过 SSL 验证的名为`insecure_skip_verify`的选项。

## 加密 <a id="secrets"></a>

未加密的信息或字典可以通过 HTTP API 或日志获得。

在Prometheus中，从服务发现中检索的元数据不被加密。在整个Prometheus系统中，数据指标不被加密。

配置文件中包含加密的字段\(在文档中已明确标记\)不会在日志中或通过 HTTP API 公开。加密的内容不应该放在其他配置字段中\(未明确标记的字段\)，因为组件通常会在其 HTTP 端点上公开其配置。

依赖所使用的其他来源的加密信息\(例如 EC2 服务发现所使用的`AWS_SECRET_KEY`环境变量\)最终可能会由于我们控制之外的代码或偶然暴露秘密存储的功能而最终暴露出来。

## 拒绝服务 <a id="denial-of-service"></a>

对于负载过高或开销太大的查询，有一些缓和措施。但是，如果提供的查询/数据指标太多或太昂贵，组件将崩溃。受信任的用户行为比恶意行为更容易出现崩溃。

用户有责任确保为组件提供足够的资源，包括 CPU、RAM、磁盘空间、IOPS、文件描述符和带宽。

建议监控所有组件的故障，并使其在故障时自动重启。

## 库 <a id="libraries"></a>

本文考虑的是从原始源代码构建的原始二进制文件。如果您修改 Prometheus 源代码或在自己的代码中使用 Prometheus 内部组件\(超出官方客户端库 API\)，则此处提供的信息不适用。

## 构建过程 <a id="build-process"></a>

Prometheus 的构建管道在 Prometheus 开发团队的许多成员以及这些提供程序的人员都可以访问的第三方提供的程序上运行。如果您担心二进制文件的确切来源，建议您自己编译构建它们，而不要依赖项目提供的预编译的二进制文件。

## 外部审计 <a id="external-audits"></a>

[CNCF](https://cncf.io/)赞助了 [curry53](https://cure53.de/) 进行的外部安全审核，审核于2018年4月至2018年6月进行。

有关更多详细信息，请阅读[审核的最终报告](https://prometheus.io/assets/downloads/2018-06-11--cure53_security_audit.pdf)。

