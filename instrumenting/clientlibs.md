# 客户端库

在监控服务之前，您需要通过 Prometheus 客户端库在其代码中添加集成。

选择与您程序编写语言相匹配的 Prometheus 客户端库。您可以通过应用程序实例上的 HTTP 端点定义和暴露内部数据指标:

* [Go](https://github.com/prometheus/client\_golang)
* [Java or Scala](https://github.com/prometheus/client\_java)
* [Python](https://github.com/prometheus/client\_python)
* [Ruby](https://github.com/prometheus/client\_ruby)

非官方的第三方客户端库:

* [Bash](https://github.com/aecolley/client\_bash)
* [C](https://github.com/digitalocean/prometheus-client-c)
* [C++](https://github.com/jupp0r/prometheus-cpp)
* [Common Lisp](https://github.com/deadtrickster/prometheus.cl)
* [Elixir](https://github.com/deadtrickster/prometheus.ex)
* [Erlang](https://github.com/deadtrickster/prometheus.erl)
* [Haskell](https://github.com/fimad/prometheus-haskell)
* [Lua for Nginx](https://github.com/knyar/nginx-lua-prometheus)
* [Lua for Tarantool](https://github.com/tarantool/prometheus)
* [.NET/C#](https://github.com/prometheus-net/prometheus-net)
* [Node.js](https://github.com/siimon/prom-client)
* [Perl](https://metacpan.org/pod/Net::Prometheus)
* [PHP](https://github.com/endclothing/prometheus\_client\_php)
* [R](https://github.com/cfmack/pRometheus)
* [Rust](https://github.com/pingcap/rust-prometheus)

当 Prometheus 采集您实例的 HTTP 端点时，客户端库会将所有跟踪指标的当前状态发送到服务器。

如果没有适用于您的语言的客户端库，或者您想避免依赖关系，您也可以自己实现一种受支持的[格式](exposition\_formats.md)来暴露数据指标。

在实现新的 Prometheus 客户端库时，请遵循[编写客户端库的准则](writing\_clientlibs.md)。请注意，本文档仍在不断变化中。也请考虑参照[开发邮件列表](https://groups.google.com/forum/#!forum/prometheus-developers)。我们很乐意就如何使您的库尽可能有用和一致提供建议。
