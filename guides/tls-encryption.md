# 使用 TLS 加密 Prometheus API 和 UI 端点

对于与 Prometheus 实例(即表达式浏览器或 [HTTP API](../prometheus/querying/api.md))的连接，Prometheus 不直接支持[传输层安全](https://en.wikipedia.org/wiki/Transport\_Layer\_Security) (TLS)加密如果您想对这些连接强制执行 TLS，建议将 Prometheus 与[反向代理](https://www.nginx.com/resources/glossary/reverse-proxy-server/)结合使用，并在代理层应用 TLS。您可以使用任何您喜欢的反向代理，在本指南中，我们将提供一个 [nginx 示例](tls-encryption.md#nginx-example)。

{% hint style="info" %}
NOTE: 尽管不支持\_到\_ Prometheus 实例 TLS 连接，但\_从\_ Prometheus 实例到[采集目标](../prometheus/configuration/configuration.md#tls\_config)的连接均支持TLS。
{% endhint %}

## nginx 示例 <a href="#nginx-example" id="nginx-example"></a>

假设您要在 `example.com` 域(您所拥有的) 上可用的 [nginx](https://www.nginx.com/) 服务后面运行 Prometheus 实例，并且所有 Prometheus 端点都可以通过 `/prometheus` 端点使用。因此 Prometheus 的 `/metrics` 端点的完整 URL 为:

```
https://example.com/prometheus/metrics
```

假设您已经使用 [OpenSSL](https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs) 或类似工具生成了以下内容:

* SSL 证书文件 `/root/certs/example.com/example.com.crt`
* SSL 密钥文件 `/root/certs/example.com/example.com.key`

您可以使用以下命令生成自签名证书和私钥:

```bash
mkdir -p /root/certs/example.com && cd /root/certs/example.com
openssl req \
  -x509 \
  -newkey rsa:4096 \
  -nodes \
  -keyout example.com.key \
  -out example.com.crt
```

在提示符下填写适当的信息，并确保在 `Common Name` 提示符下填入 `example.com`。

## nginx 配置 <a href="#nginx-configuration" id="nginx-configuration"></a>

下面是一个示例 [`nginx.conf`](https://www.nginx.com/resources/wiki/start/topics/examples/full/) 配置文件。 使用此配置，nginx将： Below is an example [`nginx.conf`](https://www.nginx.com/resources/wiki/start/topics/examples/full/) configuration file. With this configuration, nginx will:

* 使用您提供的证书和密钥实现 TLS 加密
* 将 `/prometheus` 端点的所有连接代理到在同一主机上运行的 Prometheus 服务器(同时从URL中删除 `/prometheus`)

```
http {
    server {
        listen              443 ssl;
        server_name         example.com;
        ssl_certificate     /root/certs/example.com/example.com.crt;
        ssl_certificate_key /root/certs/example.com/example.com.key;

        location /prometheus {
            proxy_pass http://localhost:9090/;
        }
    }
}

events {}
```

以 root 用户启动 nginx(因为 nginx 需要监听到 443 端口):

```bash
sudo nginx -c /usr/local/etc/nginx/nginx.conf
```

NOTE: 此示例使用 `/etc/nginx` 作为 nginx 配置文件，但这会因安装而异。其它[常用 nginx 配置目录](http://nginx.org/en/docs/beginners\_guide.html)包括 `/usr/local/nginx/conf` 和 `/usr/local/etc/nginx`。

## Prometheus 配置 <a href="#prometheus-configuration" id="prometheus-configuration"></a>

在 nginx 代理后面运行 Prometheus 时，您需要将外部 URL 设置为 `http://example.com/prometheus`，并将路由前缀设置为 `/`：

```bash
prometheus \
  --config.file=/path/to/prometheus.yml \
  --web.external-url=http://example.com/prometheus \
  --web.route-prefix="/"
```

## 测试 <a href="#testing" id="testing"></a>

如果您想使用 `example.com` 域在本地测试 nginx 代理，则可以在 `/etc/hosts` 文件中添加一个条目，以将 `example.com` 重新路由到 `localhost`:

```
127.0.0.1     example.com
```

然后，您可以使用 cURL 与本地 nginx/Prometheus 进行交互：

```bash
curl --cacert /root/certs/example.com/example.com.crt \
  https://example.com/prometheus/api/v1/label/job/values
```

您可以使用 `--insecure` 或 `-k` 标志连接到 nginx 服务器，而无需指定证书：

```bash
curl -k https://example.com/prometheus/api/v1/label/job/values
```
