---
title: 使用基本身份验证保护 Prometheus API 和 UI 端点
---

# 使用基本身份验证保护 Prometheus API 和 UI 端点

Prometheus 不直接支持与 Prometheus [表达式浏览器](https://prometheus.io/docs/visualization/browser)和 [HTTP API](https://prometheus.io/docs/prometheus/latest/querying/api) 的连接的[基本身份验证](https://en.wikipedia.org/wiki/Basic_access_authentication)\(也称为"基本身份验证"\)。如果您想对这些连接强制执行基本身份认证，建议将 Prometheus 与反向代理结合使用，并在代理层应用身份验证。您可以在 Prometheus 中使用任何您喜欢的反向代理服务。本指南中，我们将提供一个 [nginx 示例]()。

Note: 尽管_到_ Prometheus 实例的连接不支持基本身份认证，但_从_ Prometheus 实例到[抓取目标](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config)的连接均支持基本身份认证。

## nginx 示例 <a id="nginx-example"></a>

假设您要运行 Prometheus 实例作为运行在 `localhost:12321` 的 [nginx](https://www.nginx.com) 服务的后端负载，并且所有 Prometheus 端点都可以通过 `/prometheus` 端点使用。因此 Prometheus `/metrics`端点的完整 URL 为：

```text
http://localhost:12321/prometheus/metrics
```

假设您想要访问 Prometheus 实例的所有用户都需要输入用户名和密码。在此示例中，使用 `admin` 作为用户名，然后设定您所想要任何密码。

首先，使用 [`htpasswd`](https://httpd.apache.org/docs/2.4/programs/htpasswd.html) 创建一个 `.htpasswd` 文件用来保存用户名/密码，并将其保存在 `/etc/nginx` 目录中:

```bash
mkdir -p /etc/nginx
htpasswd -c /etc/nginx/.htpasswd admin
```

Note: 此示例使用 `/etc/nginx` 作为 nginx 配置文件\(包括 `.htpasswd` 文件\)的位置，但这会因安装而异。其它\[常用 nginx 配置目录\]\(\([http://nginx.org/en/docs/beginners\_guide.html\)包括](http://nginx.org/en/docs/beginners_guide.html%29包括) `/usr/local/nginx/conf` 和 `/usr/local/etc/nginx`

## nginx 配置 <a id="nginx-configuration"></a>

如下是一个示例 [`nginx.conf`](https://www.nginx.com/resources/wiki/start/topics/examples/full/) 配置文件 \(用户名密码保存在 `/etc/nginx/.htpasswd`\). 使用此配置，nginx 将对与 `/prometheus` 端点\(代理 Prometheus\)的所有连接强制执行基本身份验证：

```text
http {
    server {
        listen 12321;

        location /prometheus {
            auth_basic           "Prometheus";
            auth_basic_user_file /etc/nginx/.htpasswd;

            proxy_pass           http://localhost:9090/;
        }
    }
}

events {}
```

使用上面的配置启动nginx：

```bash
nginx -c /etc/nginx/nginx.conf
```

## Prometheus 配置 <a id="prometheus-configuration"></a>

在 nginx 代理后端运行 Prometheus 时，您需要将外部URL设置为 `http://localhost:12321/prometheus` 并将路由前缀设置为 `/`:

```bash
prometheus \
  --config.file=/path/to/prometheus.yml \
  --web.external-url=http://localhost:12321/prometheus \
  --web.route-prefix="/"
```

## 测试 <a id="testing"></a>

您可以使用 cURL 与本地 nginx/Prometheus 进行交互，尝试如下请求:

```bash
curl --head http://localhost:12321/prometheus/graph
```

这将返回 `401 Unauthorized` 响应，因为您未能提供有效的用户名和密码。该响应还将包含 nginx 提供的 `WWW-Authenticate: Basic realm="Prometheus"` 响应头, 提示 nginx `auth_basic` 参数指定的 `Prometheus` 基本认证域。

要使用基本身份验证成功访问 Prometheus 端点，如 `/metrics`,请使用`-u` 标志提供正确的用户名，并在出现提示时输入密码：

```bash
curl -u admin http://localhost:12321/prometheus/metrics
Enter host password for user 'admin':
```

然后应该返回 Prometheus 数据指标输出，看起来应该像这样：

```text
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0.0001343
go_gc_duration_seconds{quantile="0.25"} 0.0002032
go_gc_duration_seconds{quantile="0.5"} 0.0004485
...
```

## 小结 <a id="summary"></a>

在本指南中，您将用户名和密码存储在 `.htpasswd` 文件中，将 nginx 配置为使用该文件中的凭据对访问 Prometheus HTTP 端点的用户进行身份验证，启动 nginx，并为 Prometheus 配置反向代理。

