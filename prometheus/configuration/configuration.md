---
title: 配置
---

# 配置

Prometheus 通过命令行参数和配置文件进行配置。命令行参数配置了不可变的系统参数\(如存储位置，保留在磁盘和内存中的数据量等\)。配置文件定义了与采集数据[作业及实例](jobs_instances.md)相关的所有内容及[加载哪些规则文件](recording_rules.md#configuring-rules)。

执行`./prometheus -h`查看所有可用的命令行参数

Prometheus 可以在运行时重新加载其配置。如果新的配置格式不正确，则不会应用相关更改。通过向 Prometheus 进程发送`SIGHUP`信号或向`/-/reload`端点发送 HTTP POST 请求\(当启动`--web.enable-lifecycle`标志时\)来触发配置重载。同时，这也会重新加载所有已配置的规则文件。

## 配置文件

使用`--config.file`标志指定要加载的配置文件。

配置文件是 [YAML 格式](https://en.wikipedia.org/wiki/YAML)的，由以下描述的格式进行定义。方括号表示参数是可选的。对于非列表参数，该值设置为指定的默认值。

通用占位符定义如下：

* `<boolean>`: 值为 `true` 或 `false` 的布尔值
* `<duration>`: 可被正则表达式`[0-9]+(ms|[smhdwy]`匹配的一段时间
* `<labelname>`: 可被正则表达式`[a-zA-Z_][a-zA-Z0-9_]*`匹配的字符串
* `<labelvalue>`: unicode 字符串
* `<filename>`: 当前工作目录中的合法的路径
* `<host>`: 由主机名或 IP 后跟可选端口号组成的合法的字符串
* `<path>`: 合法的 URL 路径
* `<scheme>`: 字符串，可取值为 `http` 或 `https`
* `<string>`: 常规字符串
* `<secret>`: 加密后的常规字符串，例如密码
* `<tmpl_string>`: 使用模版扩展的字符串

个别其它占位符单独指出。

在[这里](https://github.com/prometheus/prometheus/blob/release-2.17/config/testdata/conf.good.yml)可以找到合法的示例配置文件。

全局配置指定在所有其它配置上下文中有效的参数，它们还用作其它配置部分的默认配置。

```yaml
global:
  # 默认采集目标数据指标的频率
  [ scrape_interval: <duration> | default = 1m ]

  # 采集请求的超时时间
  [ scrape_timeout: <duration> | default = 10s ]

  # 评估规则的频率
  [ evaluation_interval: <duration> | default = 1m ]

  # 与外部扩展系统(federation, remote storage, Alertmanager)通信时添加到时间序列或告警的标签
  external_labels:
    [ <labelname>: <labelvalue> ... ]

  # 记录 PromQL 查询的文件。重新加载配置将重新打开文件
  [ query_log_file: <string> ]

# rule_files 指定了 globs 列表，从所有匹配的文件中读取规则和告警
rule_files:
  [ - <filepath_glob> ... ]

# 数据采集的配置列表
scrape_configs:
  [ - <scrape_config> ... ]

# Alerting 指定了与 Alertmanager 相关的配置
alerting:
  alert_relabel_configs:
    [ - <relabel_config> ... ]
  alertmanagers:
    [ - <alertmanager_config> ... ]

# 与远程写相关的配置
remote_write:
  [ - <remote_write> ... ]

# 与远程读相关的配置
remote_read:
  [ - <remote_read> ... ]
```

`scrape_config` 配置部分指定了一组目标和参数，这些目标和参数描述了如何对它们进行数据采集。一般情况下，一个采集配置指定一个作业。在高级配置中，这可能会改变。

可以通过`static_configs`参数配置静态目标，也可以使用支持的服务发现机制动态发现目标。

此外，`relabel_configs`允许在数据采集之前对任何目标及其标签进行进一步修改。

```yaml
# 数据采集的作业名称
job_name: <job_name>

# 本作业采集数据的频率
[ scrape_interval: <duration> | default = <global_config.scrape_interval> ]

# 本作业数据采集的超时时间
[ scrape_timeout: <duration> | default = <global_config.scrape_timeout> ]

# 从目标采集数据指标的 HTTP 资源路径
[ metrics_path: <path> | default = /metrics ]

# honor_labels 控制 Prometheus 如何处理已包含在采集数据中的标签与 Prometheus 将在服务器端附加的标签(包含 "job" 和 "instance" 标签，手动配置的目标标签以及由服务发现实现生成的标签)之间的冲突
#
# 如果 honor_labels 设置为 "true"，则通过保留采集数据中的标签值并忽略冲突的服务器端标签来解决标签冲突。
#
# 如果 honor_labels 设置为 "false"，则通过将已采集数据中的冲突标签重命名为 "exported_<original-label>"(例如 "exported_instance"、"exported_job")，并附加服务器端标签来解决标签冲突。
#
# 将 honor_labels 设置为 "true" 对于诸如联合和采集 Pushgateway 的用例很有用，在这种情况下应保留目标中指定的所有标签。
#
# 请注意，此设置不会影响任何全局配置的 "external_labels"。在与外部系统通信时，仅当时间序列尚没有给定标签时才始终应用它们，否则将忽略它们
[ honor_labels: <boolean> | default = false ]

# honor_timestamps 控制 Prometheus 是否保留采集数据中存在的时间戳。
#
# 如果 honor_timestamps 设置为 "true"，则将使用目标暴露的数据指标的时间戳。
#
# 如果honor_timestamps设置为 "false"，则目标暴露的数据指标的时间戳将被忽略。
[ honor_timestamps: <boolean> | default = true ]

# 配置用于请求的协议
[ scheme: <scheme> | default = http ]

# 可选的 HTTP URL参数
params:
  [ <string>: [<string>, ...] ]

# 使用配置的用户名和密码在每个数据擦剂请求上设置 `Authorization` 请求头
# password 和 password_file 是互斥的。
basic_auth:
  [ username: <string> ]
  [ password: <secret> ]
  [ password_file: <string> ]

# 使用 bearer_token 在每个数据采集请求上设置 `Authorization` 请求头。它与 `bearer_token_file` 参数互斥。
[ bearer_token: <secret> ]

# 使用从 bearer_token_file 文件中读取的 bearer_token 在每个采集请求上设置 `Authorization` 请求头。它与 `bearer_tokene` 参数互斥。
[ bearer_token_file: /path/to/bearer/token/file ]

# 数据采集请求的 TLS 设置
tls_config:
  [ <tls_config> ]

# 可选的代理 URL
[ proxy_url: <string> ]

# Azure 服务发现配置列表
azure_sd_configs:
  [ - <azure_sd_config> ... ]

# Consul 服务发现配置列表
consul_sd_configs:
  [ - <consul_sd_config> ... ]

# DNS 服务发现配置列表
dns_sd_configs:
  [ - <dns_sd_config> ... ]

# EC2 服务发现配置列表
ec2_sd_configs:
  [ - <ec2_sd_config> ... ]

# OpenStack服务发现配置列表
openstack_sd_configs:
  [ - <openstack_sd_config> ... ]

# 文件服务发现配置列表
file_sd_configs:
  [ - <file_sd_config> ... ]

# GCE 服务发现配置列表
gce_sd_configs:
  [ - <gce_sd_config> ... ]

# Kubernetes 服务发现配置列表
kubernetes_sd_configs:
  [ - <kubernetes_sd_config> ... ]

# Marathon 服务发现配置列表
marathon_sd_configs:
  [ - <marathon_sd_config> ... ]

# AirBnB Nerve 服务发现配置列表
nerve_sd_configs:
  [ - <nerve_sd_config> ... ]

# Zookeeper 服务发现配置列表
serverset_sd_configs:
  [ - <serverset_sd_config> ... ]

# Triton 服务发现配置列表
triton_sd_configs:
  [ - <triton_sd_config> ... ]

# 静态配置目标列表
static_configs:
  [ - <static_config> ... ]

# 目标重新标记配置列表
relabel_configs:
  [ - <relabel_config> ... ]

# 数据指标重新标记配置列表
metric_relabel_configs:
  [ - <relabel_config> ... ]

# 每次采集将接受的采集样本数限制。
# 如果在数据指标重新标记后存在的样本数量操作此数量，则本次数据采集将被视为不合格。0 表示没有限制
[ sample_limit: <int> | default = 0 ]
```

tls\_config 配置 TLS 连接相关内容。

```yaml
# 用于验证 API 服务器证书的 CA 证书
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

azure 服务发现配置允许从 Azure VMs 发现数据收集的目标。

重新标记过程中，支持以下的 meta 标签：

* `__meta_azure_machine_id`: 设备 ID
* `__meta_azure_machine_location`: 设备运行位置
* `__meta_azure_machine_name`: 设备名称
* `__meta_azure_machine_os_type`: 设备操作系统
* `__meta_azure_machine_private_ip`: 设备的私有 IP 地址
* `__meta_azure_machine_public_ip`: 设备的公有 IP 地址\(如果存在\)
* `__meta_azure_machine_resource_group`: 设备的资源组
* `__meta_azure_machine_tag_<tagname>`: 设备的标签值
* `__meta_azure_machine_scale_set`: VM 所属的伸缩集的名称\(当您使用此伸缩集时才设置此值\)
* `__meta_azure_subscription_id`: ~~订阅 ID~~
* `__meta_azure_tenant_id`: 租户 ID

请参阅以下有关 Azure 服务发现的配置项：

```yaml
# 访问 Azure API 信息
# Azure 环境
[ environment: <string> | default = AzurePublicCloud ]

# 身份验证方式，OAuth 或 ManagedIdentity.
# 参见 https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview
[ authentication_method: <string> | default = OAuth]
# subscription ID. 必需
subscription_id: <string>
# 可选的 tenant ID. 当认证方式为 OAuth 时必需
[ tenant_id: <string> ]
# 可选的 client ID. 当认证方式为 OAuth 时必需
[ client_id: <string> ]
# 可选的 client secret.当认证方式为 OAuth 时必需
[ client_secret: <secret> ]

# 重读实例列表的刷新间隔
[ refresh_interval: <duration> | default = 300s ]

# 采集数据的端口。如果使用公有 IP 地址，则必须在重新标记规则中指定该地址。
[ port: <int> | default = 80 ]
```

consul 服务发现配置允许从 [consul](https://www.consul.io/) 的 Catalog API 中自动发现采集数据的目标。

重新标记过程中，支持以下的 meta 标签：

* `__meta_consul_address`: 目标地址
* `__meta_consul_dc`: 目标数据中心名称
* `__meta_consul_tagged_address_<key>`: 节点标记的目标地址的键值
* `__meta_consul_metadata_<key>`: 节点标记的目标元数据的键值
* `__meta_consul_node`: 目标的节点名称
* `__meta_consul_service_address`: 目标的服务地址
* `__meta_consul_service_id`: 目标的服务 ID
* `__meta_consul_service_metadata_<key>`: 目标服务的元数据键值
* `__meta_consul_service_port`: 目标服务端口
* `__meta_consul_service`: 目标所属服务的名称
* `__meta_consul_tags`: 标签分割符连接的目标的标签列表

  \`\`\`yaml

  **Consul API 的访问信息。根据 Consul 的需要进行定义**

  \[ server:  \| default = "localhost:8500" \]

  \[ token:  \]

  \[ datacenter:  \]

  \[ scheme:  \| default = "http" \]

  \[ username:  \]

  \[ password:  \]

tls\_config: \[  \]

## 待发现数据采集的目标服务列表。如果省略，则采集所有服务

services: \[ -  \]

## 查看 [https://www.consul.io/api/catalog.html\#list-nodes-for-service](https://www.consul.io/api/catalog.html#list-nodes-for-service) 进一步了解可以使用的过滤器

## 可选的标签列表，用于过滤给定服务的节点。服务必须包含列表中的所有标签

tags: \[ -  \]

## node\_meta 用来过滤给定服务的节点

\[ node\_meta: \[ :  ... \] \]

## 将 Consul 标签连接到标签的字符串

\[ tag\_separator:  \| default = , \]

## 允许过期的结果\(参阅 [https://www.consul.io/api/features/consistency.html\).将减少](https://www.consul.io/api/features/consistency.html%29.将减少) Consul 的负载.

\[ allow\_stale:  \]

## 服务发现名称刷新时间

## 在大型架构中，增加此值可能是个好主意，因为 catalog 不断更改

\[ refresh\_interval:  \| default = 30s \]

```text
请注意，用于数据采集的目标的 IP 地址和端口被组合为`<__meta_consul_address>:<__meta_consul_service_port>`。 但是，在某些 Consul 设置中，相关地址在`__meta_consul_service_address`中。在这种情况下，您可以使用重新标记功能来替换特殊的`__address__`标签。

重新标记阶段是基于任意标签为服务筛选服务或节点的首选且功能更强大的方法。对于拥有数千项服务的用户而言，直接使用 Consul API 更为有效，该 API 具有基本的过滤节点支持(一般通过节点元数据和单个标签）。


## <dns_sd_config>
TODO:
## <ec2_sd_config>
TODO:
## <openstack_sd_config>
TODO:
## <file_sd_config>
TODO:
## <gce_sd_config>
TODO:

## <kubernetes_sd_config>
kubernetes 服务发现配置允许从 [Kubernetes](https://kubernetes.io/) 的 REST API 发现数据采集的目标，并始终与集群状态保持同步。

可以配置以下 `role` 角色类型之一来发现目标。

### `node`
`node`角色为集群每个节点发现一个目标，其地址默认为 Kubelet 的 HTTP 端口。目标地址默认为 Kubernetes 节点对象按照`NodeInternalIP`，`NodeInternalIP`，`NodeExternalIP`，`NodeLegacyHostIP`和`NodeHostName`的顺序检索的第一个现有地址。

可用的 meta 标签：
- `__meta_kubernetes_node_name`: 节点对象名称
- `__meta_kubernetes_node_label_<labelname>`: 节点对象的标签
- `__meta_kubernetes_node_annotation_<annotationname>`: 节点对象的注解
- `__meta_kubernetes_node_annotationpresent_<annotationname>`: 对于来自节点对象的每个注解，为true
- `__meta_kubernetes_node_address_<address_type>`: 节点对象指定地址类型的第一个地址(如果存在)

另外，该节点的`instance`标签将设置为从 API 服务器检索到的节点名称。

### `service`
`service`角色发现每个服务及其端口的数据采集目标。通常，这对于黑盒监控服务很有用。地址将设置为服务的 Kubernetes DNS 名称及相应的服务端口。

可用的 meta 标签：
- `__meta_kubernetes_namespace`: service 对象的名称空间
- `__meta_kubernetes_service_annotation_<annotationname>`: service 对象的注解
- `__meta_kubernetes_service_annotationpresent_<annotationname>`: 对于每个 service 对象的注解为 true
- `__meta_kubernetes_service_cluster_ip`: service 的 ClusterIP 地址(不适用于 ExternalName 类型的 service)
- `__meta_kubernetes_service_external_name`: service 的 DNS 名称(适用于 ExternalName 类型的 services)
- `__meta_kubernetes_service_label_<labelname>`: service 对象的标签
- `__meta_kubernetes_service_labelpresent_<labelname>`: 对于每个 service 对象的标签为 true
- `__meta_kubernetes_service_name`: service 对象的名称
- `__meta_kubernetes_service_port_name`: 目标的服务端口名称
- `__meta_kubernetes_service_port_protocol`: 目标的服务端口协议
- `__meta_kubernetes_service_type`: service 的类型

### `pod`
`pod`角色发现所有 pod 并将其暴露为目标。对于容器的每个声明的端口，将生成一个目标。如果容器没有指定的端口，则会为每个容器创建无端口目标，以通过重新标记手动添加端口。

可用的 meta 标签：
- `__meta_kubernetes_namespace`: pod 对象的名称空间
- `__meta_kubernetes_pod_name`: pod 对象名称.
- `__meta_kubernetes_pod_ip`: pod 对象的 IP
- `__meta_kubernetes_pod_label_<labelname>`: pod 对象的标签
- `__meta_kubernetes_pod_labelpresent_<labelname>`: 如果是 pod 对象的标签，则为 true
- `__meta_kubernetes_pod_annotation_<annotationname>`: pod 对象的注解
- `__meta_kubernetes_pod_annotationpresent_<annotationname>`: 如果是 pod 对象的注解，则为 true
- `__meta_kubernetes_pod_container_init`: 如果容器是初始化容器，则为 true
- `__meta_kubernetes_pod_container_name`: 目标地址指向的容器的名称
- `__meta_kubernetes_pod_container_port_name`: 容器的端口名称
- `__meta_kubernetes_pod_container_port_number`: 容器的端口号
- `__meta_kubernetes_pod_container_port_protocol`: 容器端口协议
- `__meta_kubernetes_pod_ready`: pod 是否已经就绪
- `__meta_kubernetes_pod_phase`: pod 的生命周期，可能的值为 Pending, Running, Succeeded, Failed 或 Unknown
- `__meta_kubernetes_pod_node_name`: pod 被调度到的节点名称
- `__meta_kubernetes_pod_host_ip`: pod 对象的当前主机 IP
- `__meta_kubernetes_pod_uid`: pod 对象的 UID
- `__meta_kubernetes_pod_controller_kind`: pod 控制器类型
- `__meta_kubernetes_pod_controller_name`: pod 控制器名称

### `endpoints`
`endpoints`角色从列出的服务的 endpoints 中发现目标。对于每个 endpoints 地址，每个端口都发现一个目标。如果 endpoints 后端负载为 pod，则该 pod 的所有其它未绑定到 endpoints 端口的容器端口也将被发现为目标。

可用的 meta 标签：
- `__meta_kubernetes_namespace`: endpoints 对象的名称空间
- `__meta_kubernetes_endpoints_name`: endpoints 对象的名称
- 对于直接从 endpoints 列表中发现的所有目标(不是从 pod 获取的)，将附加以下标签：
    - `__meta_kubernetes_endpoint_hostname`: endpoint 的主机名
    - `__meta_kubernetes_endpoint_node_name`: endpoint 节点名称
    - `__meta_kubernetes_endpoint_ready`: endpoint 是否已经就绪
    - `__meta_kubernetes_endpoint_port_name`: endpoint 端口名称
    - `__meta_kubernetes_endpoint_port_protocol`: endpoint 端口协议
    - `__meta_kubernetes_endpoint_address_target_kind`: endpoint 地址类型
    - `__meta_kubernetes_endpoint_address_target_name`: endpoint 地址名称
- 如果 endpoints 归属于某个 service，则`role: service`服务发现的所有标签都适用
- 如果 endpoints 后端负载为 pod，则`role: pod`服务发现的所有标签都适用

### `ingress`
`ingress`角色发现每个入口的每个路径目标。这通常对于入口的黑盒监控很有用。地址将设置为 ingress 规范中指定的主机地址(`ingress.spec.host`)

可用的 meta 标签：
- `__meta_kubernetes_namespace`: ingress 对象的名称空间
- `__meta_kubernetes_ingress_name`: ingress 对象的名称.
- `__meta_kubernetes_ingress_label_<labelname>`: ingress 对象的标签.
- `__meta_kubernetes_ingress_labelpresent_<labelname>`: 如果是 ingress 对象的标签，则为 true
- `__meta_kubernetes_ingress_annotation_<annotationname>`: ingress 对象的注解
- `__meta_kubernetes_ingress_annotationpresent_<annotationname>`: 如果是 ingress 对象的注解，则为 true
- `__meta_kubernetes_ingress_scheme`: ingress 的协议。如果配置了 TLS，则为 `https`。默认为 `http`。
- `__meta_kubernetes_ingress_path`: ingress 规范中指定的路径(`ingress.spec.path`).默认为 `/`.

请参阅如下 Kubernetes 服务发现配置项：
```yaml
# 访问 Kubernetes API 的信息 

# apiserver 地址。如果为空，Prometheus 被假定为在集群内部运行，将自动发现 apiserver，并使用位于 /var/run/secrets/kubernetes.io/serviceaccount/ 的文件作为 pod 的 CA 证书和令牌文件 
[ api_server: <host> ]

# 应该被发现的实例的 Kubernetes 角色
role: <role>

# 用于向 apiserver 进行身份认证的可选身份认证信息
# 请注意，`basic_auth`, `bearer_token` 和 `bearer_token_file`选项是互斥的
# `password`和`password_file`是互斥的

# 可选的 HTTP basic_auth 基本身份认证信息
basic_auth:
  [ username: <string> ]
  [ password: <secret> ]
  [ password_file: <string> ]

# 可选的 bearer_token 承载令牌身份认证信息
[ bearer_token: <secret> ]

# 可选的 bearer_token_file 承载令牌文件身份认证信息
[ bearer_token_file: <filename> ]

# 可选的代理 URL
[ proxy_url: <string> ]

# TLS 相关配置.
tls_config:
  [ <tls_config> ]

# 可选的名称空间发现。如果省略，则使用所有名称空间
namespaces:
  names:
    [ - <string> ]

# 可选的标签和字段选择器，用于限制服务发现为可用资源的子集
# 请参阅 https://kubernetes.io/docs/concepts/overview/working-with-objects/field-selectors/ 和  https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ 进一步了解有关可以使用的过滤器的信息
# Endpoints role 支持 pod, service 和 endpoints 选择器, 其它 roles 仅支持与 role 本身匹配的选择器(如 node role 只支持 node 选择器).

# 请注意，在决定使用字段/标签选择器时，请确保这是最好的方法 - 
# 它将阻止 Prometheus 对所有数据采集配置重用单个  LIST/WATCH 权限。这可能会给 Kubernetes API 带来更大的负担，因为每个选择器组合都会有额外的的 LIST/WATCH 权限。
# 另一方面，如果您只想监视大型集群中的一小部分 Pod，建议使用选择器。
# 根据具体情况决定是否使用选择器。
[ selectors:
  [ - role: <role>  
    [ label: <string> ]
    [ field: <string> ] ]]
```

`role`必须是`endpoints`, `service`, `pod`, `node` 或`ingress`

有关为 Kubernetes 配置 Prometheus 的详细示例，请参见 [Prometheus 示例配置文件](https://github.com/prometheus/prometheus/blob/release-2.17/documentation/examples/prometheus-kubernetes.yml)

您可能希望查看第三方的 [Prometheus Operator](https://github.com/coreos/prometheus-operator)，它可以在 Kubernetes 上自动设置 Prometheus。

TODO:

TODO:

TODO:

TODO:

`static_config`允许指定目标列表和目标的通用标签集。这是在数据采集配置中指定静态目标的典型方法。

```yaml
# static config 配置指定的目标列表
targets:
  [ - '<host>' ]

# 为从目标中采集的数据分配标签
labels:
  [ <labelname>: <labelvalue> ... ]
```

重新标记是一个可以在数据采集之前动态重写目标标签的强大工具。每个数据采集配置可以配置多个重新标记步骤。他们按照在配置文件中出现的顺序依次应用于目标的标签集。

最初，除了已配置的每个目标标签外，目标的`job`标签设置为各个数据采集配置的`job_name`的标签值。`__address__`标签设置为目标的`<host>:<port>`地址。重新标记后，如果在重新标记期间未设置`instance`标签，则默认将其设置为`__address__`的值。`__scheme__`和`__metrics_path__`标签将被分别设置为数据采集目标的协议和路径。`__param_<name>`标签设置为传递的 URL 参数中第一个为`<name>`的值。

在重新标记阶段，可能会加上以`__meta_`开头的其它标签。它们由提供目标服务发现的方法设置，并在各个方法之间有所不同。

重新标记完成后，`__`开头的标签将从标签集中删除。

如果重新标记过程中仅需要临时存储标签值\(作为后续重新标记步骤的输入\)，请使用`__tmp`标签名称前缀。 保证该前缀不会被 Prometheus 单独使用。

```yaml
# 源标签从现有标签中选择值。他们的内容使用配置的分隔符连接起来，并与配置的正则表达式匹配，以进行替换、保持和删除动作
[ source_labels: '[' <labelname> [, ...] ']' ]

# 在连接的源标签值之间放置分隔符
[ separator: <string> | default = ; ]

# 在替换操作后，将结果值写入的标签
# 如果是替换操作，该选项是必需的. 正则表达式捕获组可用
[ target_label: <labelname> ]

# 与提取的值匹配的正则表达式
[ regex: <regex> | default = (.*) ]

# 取源标签值哈希值的模数
[ modulus: <uint64> ]

# 执行正则表达式替换的替换值。正则表达式捕获组可用。
[ replacement: <string> | default = $1 ]

# 基于正则表达式匹配执行的操作
[ action: <relabel_action> | default = replace ]
```

`<regex>`可以是任何有效的 [RE2 正则表达式](https://github.com/google/re2/wiki/Syntax)。`replace`, `keep`, `drop`, `labelmap`,`labeldrop`和`labelkeep`等操作是必需的。正则表达式被锚定在两端，要取消锚定正则表达式，请使用`.*<regex>.*`。

`<reloabel_action>`确定要执行的重新标记操作：

* `replace`: 将正则表达式与串联的`source_labels`匹配。然后，用`replacement`中的匹配组引用\(`${1}`,`${2}`...\)将`target_label`的值设置为`replacement`的值。如果正则表达式不匹配，则不会进行替换。
* `keep`: 删除目标的`source_labels`不能匹配`regex`的目标
* `drop`: 删除目标的`source_labels`能匹配`regex`的目标
* `hashmod`: 将`target_label`设置为串联的`source_label`的哈希的`modulus`
* `labelmap`: 将`regex`与所有标签名称匹配。然后将匹配标签的值复制到`replacement`中给定的标签名称，并用`replacement`中的匹配组引用\(`${1}`,`${2}`...\)替换它们的值。
* `labeldrop`: 将`regex`与所有标签名称匹配。任何匹配的标签将从标签集中删除。
* `labelkeep`: 将`regex`与所有标签名称匹配。任何不匹配的标签将从标签集中删除。

请谨慎使用`labeldrop`和`labelkeep`以确保一旦删除标签，数据指标仍可以被唯一标记。

对数据样本进行重新标记是入库前的最后一步。它与目标重新标记有相同的配置格式和操作。指标重新标记不适用于自动生成的时间序列，如`up`.

这样做的一种用途是将由于过多开销的而无法入库的时间序列列如黑名单。

告警重新标记应用于发送告警到 Alertmanager 之前。它与目标重新标记有相同的配置格式和操作。在扩展标记之后应用告警重新标记。

一种用途是确保具有不同扩展标签的一对高可用 Prometheus 发送相同的告警。

`alertmanager_config`部分指定了 Prometheus 向其发送告警的 Alertmanager 实例。它还提供了用于配置如何与 Alertmanager 通信的参数。

Alertmanager 可以通过`static_configs`参数静态配置，也可以使用支持的服务发现机制动态发现。

另外，`relabel_configs`允许从发现的实例中选择 Alertmanagers，并提供对使用的 API 路径的高级修改，该路径通过`__alerts_path__`标签暴露。

```yaml
# 推送告警的超时时间
[ timeout: <duration> | default = 10s ]

# Alertmanager 的 api 版本号
[ api_version: <version> | default = v1 ]

# 推送告警的 HTTP 路径前缀
[ path_prefix: <path> | default = / ]

# 配置用于请求的协议
[ scheme: <scheme> | default = http ]

# 使用配置的用户名和密码在每个请求上设置 `Authorization` 请求头
# password 和 password_file 是互斥的。
basic_auth:
  [ username: <string> ]
  [ password: <string> ]
  [ password_file: <string> ]

# 使用 bearer_token 在每个请求上设置 `Authorization` 请求头。它与 `bearer_token_file` 参数互斥。
[ bearer_token: <string> ]

# 使用从 bearer_token_file 文件中读取的 bearer_token 在每个请求上设置 `Authorization` 请求头。它与 `bearer_tokene` 参数互斥。
[ bearer_token_file: /path/to/bearer/token/file ]

# 采集请求的 TLS 设置
tls_config:
  [ <tls_config> ]

# 可选的代理 URL
[ proxy_url: <string> ]

# Azure 服务发现配置列表
azure_sd_configs:
  [ - <azure_sd_config> ... ]

# Consul 服务发现配置列表
consul_sd_configs:
  [ - <consul_sd_config> ... ]

# DNS 服务发现配置列表
dns_sd_configs:
  [ - <dns_sd_config> ... ]

# EC2 服务发现配置列表
ec2_sd_configs:
  [ - <ec2_sd_config> ... ]

# 文件服务发现配置列表
file_sd_configs:
  [ - <file_sd_config> ... ]

# GCE 服务发现配置列表
gce_sd_configs:
  [ - <gce_sd_config> ... ]

# Kubernetes 服务发现配置列表
kubernetes_sd_configs:
  [ - <kubernetes_sd_config> ... ]

# Marathon 服务发现配置列表
marathon_sd_configs:
  [ - <marathon_sd_config> ... ]

# AirBnB 的 Nerve 服务发现配置列表
nerve_sd_configs:
  [ - <nerve_sd_config> ... ]

# Zookeeper 服务发现配置列表
serverset_sd_configs:
  [ - <serverset_sd_config> ... ]

# Triton 服务发现配置列表
triton_sd_configs:
  [ - <triton_sd_config> ... ]

# 静态配置 Alertmanager 列表
static_configs:
  [ - <static_config> ... ]

# Alertmanager 重新标记配置列表
relabel_configs:
  [ - <relabel_config> ... ]
```

`write_relabel_configs`在将样本数据发送到远程节点之前重新标记样本数据。在扩展标记之后应用写重新标记。这可以用来限制发送哪些样本数据。

这里有一个[样例](https://github.com/prometheus/prometheus/blob/release-2.17/documentation/examples/remote_storage)演示了如何使用此功能。

```yaml
# 发送数据的远程端点 URL
url: <string>

# 远程写请求的超时时间
[ remote_timeout: <duration> | default = 30s ]

# 远程写过程重新标记配置列表
write_relabel_configs:
  [ - <relabel_config> ... ]

# 使用配置的用户名和密码在每个远程写请求上设置 `Authorization` 请求头
# password 和 password_file 是互斥的。
basic_auth:
  [ username: <string> ]
  [ password: <string> ]
  [ password_file: <string> ]

# 使用 bearer_token 在每个远程写请求上设置 `Authorization` 请求头。它与 `bearer_token_file` 参数互斥。.
[ bearer_token: <string> ]

# 使用从 bearer_token_file 文件中读取的 bearer_token 在每个远程写请求上设置 `Authorization` 请求头。它与 `bearer_tokene` 参数互斥。
[ bearer_token_file: /path/to/bearer/token/file ]

# 远程写请求的 TLS 设置
tls_config:
  [ <tls_config> ]

# 可选的代理 URL
[ proxy_url: <string> ]

# 配置用于写入远程存储的队列
queue_config:
  # 在阻塞从 WAL 读取更多样本之前，每个分片要缓冲的样本数。建议在每个分片中都具有足够的容量以缓冲多个请求，以在处理偶尔的慢速远程请求时保持吞吐量。
  [ capacity: <int> | default = 500 ]
  # 分片的最大数量，即并发数量
  [ max_shards: <int> | default = 1000 ]
  # 分片的最小数量，即并发数量
  [ min_shards: <int> | default = 1 ]
  # 每次发送的最大样本数
  [ max_samples_per_send: <int> | default = 100]
  # 样本在缓冲区中等待的最长时间
  [ batch_send_deadline: <duration> | default = 5s ]
  # 初始重试延迟。每次重试都会加倍
  [ min_backoff: <duration> | default = 30ms ]
  # 最大重试延迟
  [ max_backoff: <duration> | default = 100ms ]
```

这里是有此功能的[集成列表](integrations.md#remote-endpoints-and-storage)

```yaml
# 查询数据的远程端点 URL
url: <string>

# 可选的相等性匹配器列表，必须在选择器中提供该列表以查询远程读取端点
required_matchers:
  [ <labelname>: <labelvalue> ... ]

# 远程写请求的超时时间
[ remote_timeout: <duration> | default = 30s ]

# 是否对本地存储应具有完整数据的时间范围的查询进行读取
[ read_recent: <boolean> | default = false ]

# 使用配置的用户名和密码在每个远程写请求上设置 `Authorization` 请求头
# password 和 password_file 是互斥的。
basic_auth:
  [ username: <string> ]
  [ password: <string> ]
  [ password_file: <string> ]

# 使用 bearer_token 在每个远程写请求上设置 `Authorization` 请求头。它与 `bearer_token_file` 参数互斥。.
[ bearer_token: <string> ]

# 使用从 bearer_token_file 文件中读取的 bearer_token 在每个远程写请求上设置 `Authorization` 请求头。它与 `bearer_tokene` 参数互斥。
[ bearer_token_file: /path/to/bearer/token/file ]

# 远程写请求的 TLS 设置
tls_config:
  [ <tls_config> ]

# 可选的代理 URL
[ proxy_url: <string> ]
```

这里是有此功能的[集成列表](integrations.md#remote-endpoints-and-storage)

