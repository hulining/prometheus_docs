---
title: 数据模型
---

# 数据模型

Prometheus 基本上将所有数据存储为时间序列：属于同一数据指标和同一组标注维度的带有时间戳的数据流。除了存储的时间序列之外，Prometheus 可能会生成临时派生的时间序列作为查询的结果。

## 指标名称和标签 <a id="metric-names-and-labels"></a>

每个时间序列都由其_名称_和被称为_标签_的可选键值对唯一标识。

指标名称指定了所监测系统的一般功能\(如`http_requests_total` - 收到 HTTP 请求总数\)。它可能包含 ASCII 字符，数字下划线和冒号。它必须能被正则表达式`[a-zA-Z_:][a-zA-Z0-9_:]*`匹配。

注意：冒号是为用户自定义的规则保留的。exporter 或直接展示的组件都不应使用它们。

标签可以使 Prometheus 支持多维度数据模型：具有相同数据指标名称的标签的任何给定组合都可以标识该数据指标的特定维度实例\(如，使用`POST`方法到`/api/tracks`处理程序的所有 HTTP 请求\)。查询语言允许基于这些维度进行过滤和聚合。更改任何标签值，包括添加或删除，都会创建一个新的时间序列。

标签名称可以包含 ASCII 字符，数字和下划线。它必须能被正则表达式`[a-zA-Z_:][a-zA-Z0-9_:]*`匹配。以`__`开头的标签名称保留供内部使用。

标签值可以包含任何 Unicode 字符。

标签值为空的标签被认为等同于不存在的标签。

另请参阅[命名指标和标签的最佳实践](../practices/naming.md)

## 数据样本 <a id="samples"></a>

数据样本构成实际的时间序列数据。每个数据样本包括：

* 64 位浮点型数值
* 毫秒精度的时间戳

## 表示方式 <a id="notation"></a>

数据指标指定了数据指标名称和一组标签，通常使用如下表示方式来标识时间序列：

```text
<metric name>{<label name>=<label value>, ...}
```

例如，包含数据指标名称`api_http_requests_total`和标签`method="POST""`，`handler="/messages"`的时间序列可以这样写：

```text
api_http_requests_total{method="POST", handler="/messages"}
```

这与 [OpenTSDB](http://opentsdb.net/) 使用的表示方式相同。

