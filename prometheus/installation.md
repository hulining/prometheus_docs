---
title: 安装
---

# 安装

## 使用预编译的二进制文件

我们为大多数官方 Prometheus 组件提供了预编译的二进制文件。 请查看[下载部分](https://prometheus.io/download)以获取所有可用版本的列表。

## 从源码安装

要从源代码构建 Prometheus 组件，请参见相应代码仓库中的`Makefile`文件。

## 使用 Docker

所有 Prometheus 服务都可以作为在 [Quay.io](https://quay.io/repository/prometheus/prometheus) 或 [Docker Hub](https://hub.docker.com/r/prom/prometheus/) 上的 Docker 镜像使用。

在 Docker 上运行 Prometheus 与运行`docker run -p 9090:9090 prom/prometheus`一样简单。这将以示例配置启动 Prometheus，并将其暴露在 9090 端口上。

Prometheus 镜像使用卷来存储实际数据指标。对于生产环境的部署，强烈建议使用[数据卷容器](https://docs.docker.com/engine/admin/volumes/volumes/)模式来简化 Prometheus 升级过程中的数据管理。

想要使用您自己的配置，可以有多种选择。下面是两个示例。

### 挂载存储卷

通过运行以下命令从主机挂载您的`prometheus.yml`:

```bash
docker run \
    -p 9090:9090 \
    -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
```

或使用一个独立的卷进行配置：

```bash
docker run \
    -p 9090:9090 \
    -v /path/to/config:/etc/prometheus \
    prom/prometheus
```

### 自定义镜像

为了避免在主机上管理文件并将其绑定，可以将配置打包到镜像中。如果配置本身是静态的，并且在所有环境中都相同，则此方法很好。

为此，在一个新目录中，创建 Prometheus 配置文件`prometheus.yml`和如下 `Dockerfile`

```text
FROM prom/prometheus
ADD prometheus.yml /etc/prometheus
```

然后构建镜像并运行它：

```bash
docker build -t my-prometheus .
docker run -p 9090:9090 my-prometheuss
```

更高级的做法是在启动时使用某些工具动态渲染配置文件，甚至让守护程序定期更新它。

## 使用配置管理系统

如果您想要使用配置管理系统，则可能对以下第三方贡献工具感兴趣：

### Ansible

* [Cloud Alchemy/ansible-prometheus](https://github.com/cloudalchemy/ansible-prometheus)

### Chef

* [rayrod2030/chef-prometheus](https://github.com/rayrod2030/chef-prometheus)

### Puppet

* [puppet/prometheus](https://forge.puppet.com/puppet/prometheus)

### SaltStack

* [saltstack-formulas/prometheus-formula](https://github.com/saltstack-formulas/prometheus-formula)

