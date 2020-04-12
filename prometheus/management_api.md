---
title: 管理 API
---

# 管理 API

Prometheus 提供了一组管理 API，以简化自动化和集成。

### 健康检查 <a id="health-check"></a>

```text
GET /-/healthy
```

该端点返回 200，且检查 Prometheus 的运行状况。

### 就绪状态检测 <a id="readiness-check"></a>

```text
GET /-/ready
```

当 Prometheus 准备服务流量\(如，响应查询\)时，此端点返回 200。

### 配置重载 <a id="reload"></a>

```text
PUT  /-/reload
POST /-/reload
```

该端点触发 Prometheus 配置和规则文件的重新加载。默认情况下它是禁用的，可以通过`--web.enable-lifecycle`标志启用。

另外，可以发送`SIGHUP`信号到 Prometheus 进程来触发配置重载。

### 退出 <a id="quit"></a>

```text
PUT  /-/quit
POST /-/quit
```

该端点触发 Prometheus 的正常关闭。默认情况下它是禁用的，可以通过`--web.enable-lifecycle`标志启用。

另外，可以发送`SIGTERM`信号到 Prometheus 进程来触发正常关闭。

