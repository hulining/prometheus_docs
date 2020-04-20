---
title: 管理 API
sort_rank: 9
---

# 管理 API

Alertmanager 提供了一组管理API，以简化自动化和集成。

## 健康检查 <a id="health-check"></a>

```text
GET /-/healthy
```

该端点始终返回200，应用于检查 Alertmanager 的运行状况。

## 就绪状态检查 <a id="readiness-check"></a>

```text
GET /-/ready
```

当 Alertmanager 准备服务流量\(即响应查询\)时，此端点返回200

## 重载配置文件 <a id="reload"></a>

```text
POST /-/reload
```

该端点触发 Alertmanager 重新加载配置文件

触发配置重新加载的另一种方法是向 Alertmanager 进程发送一个`SIGHUP`信号。

