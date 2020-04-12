---
title: 运算符
---

# 运算符

## 二元运算符 <a id="binary-operators"></a>

Prometheus 的查询语言支持基本的逻辑运算和算术运算。对于两个瞬时向量之间的操作，[匹配行为](operators.md#vector-matching)可以被改变。

### 二元算术运算符号 <a id="arithmetic-binary-operators"></a>

Prometheus 中存在以下二元算术运算符：

* `+` 加
* `-` 减
* `*` 乘
* `/` 除
* `%` 取模
* `^` 乘方，幂

二元运算操作符定义在标量与标量、向量与标量和向量与向量之间。

**在两个标量之间**，其结果是显而易见的：它们评估另一个标量，运算应用于两个标量操作数的结果

**在瞬时向量和标量之间**，运算将应用于向量中每个数据样本的值。例如，如果将时间序列瞬时向量乘以 2 ，则结果是原始向量的每个样本值都乘以 2 的另一个向量。

**在两个即时向量之间**，将二元算术运算符应用于左侧向量中的每个条目，并将其应用于右侧向量中的[匹配元素](operators.md#vector-matching)。结果被传播到结果向量中，并且分组标签成为输出标签集。 数据指标名称被删除。 在右侧向量中找不到匹配条目的条目不是结果的一部分。

### 二元比较运算符 <a id="comparison-binary-operators"></a>

Prometheus 中存在以下二元比较运算符：

* `==` 等于
* `!=` 不等于
* `>`  大于
* `<`  小于
* `>=` 大于等于
* `<=` 小于等于

比较运算符定义在标量与标量、向量与标量和向量与向量之间。默认情况下，它们用于过滤。可以通过在运算符后提供`bool`来改变其行为，该布尔值将返回`0`或`1`而不是过滤。

**在两个标量之间**，必须提供`bool`修饰符，并且这些运算符会产生另一个标量`0`\(`false`\)或`1`\(`true`\)，具体取决于比较结果。

**在瞬时向量和标量之间**，运算将应用于向量中每个数据样本的值，并将比较结果为假的向量元素从结果向量中删除。如果提供了`bool`修饰符，则要删除的矢量元素的值为`0`，要保留的矢量元素的值为`1`。

**在两个即时向量之间**，运算符默认情况下充当过滤器，应用于匹配的条目。表达式不正确或在表达式另一侧找不到匹配项的向量元素将从结果中删除，而其他元素则传播到结果向量中，且分组标签成为输出标签集。如果提供了`bool`修饰符，将被丢弃的矢量元素取值为`0`，将保留的矢量元素取值为`1`，分组标签称为输出标签集。

### 逻辑/集合运算符 <a id="logical-set-binary-operators"></a>

这些二元逻辑/集合运算符仅在即时向量之间定义：

* `and` 交集
* `or`  并集
* `unless` 补集

`vector1 and vector2`产生一个由`vector1`与`vector2`具有完全匹配的标签集的元素组成的向量。其他元素被删除。度量标准名称和值从左侧向量继承。

`vector1 or vector2`产生一个包含`vector1`的所有原始元素\(标签集+值\)以及`vector2`在`vector1`中没有匹配的标签集所有元素的向量。

`vector1 unless vector2`产生一个由`vector1`在`vector2`中没有完全匹配的标签集的元素组成的向量。两个向量中的所有匹配元素都将被删除

## 矢量匹配 <a id="vector-matching"></a>

向量之间的运算会尝试在左侧向量中为左侧的每个条目找到匹配的元素。匹配行为有两种基本类型：一对一和多对一/一对多。

### 一对一向量匹配 <a id="one-to-one-vector-matches"></a>

**一对一**从运算符每側查找一对唯一的条目。在默认情况下，遵循`vector1 <operator> vector2`格式。如果两个条目具有完全相同的一组标签和相应的值，则它们匹配。`ignoring`关键字允许在匹配时忽略某些标签，而`on`关键字允许将所考虑的标签集减少为仅考虑提供的列表中的标签集：

```text
<vector expr> <bin-op> ignoring(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) <vector expr>
```

如样本数据

```text
method_code:http_errors:rate5m{method="get", code="500"}  24
method_code:http_errors:rate5m{method="get", code="404"}  30
method_code:http_errors:rate5m{method="put", code="501"}  3
method_code:http_errors:rate5m{method="post", code="500"} 6
method_code:http_errors:rate5m{method="post", code="404"} 21

method:http_requests:rate5m{method="get"}  600
method:http_requests:rate5m{method="del"}  34
method:http_requests:rate5m{method="post"} 120
```

查询示例：

```text
method_code:http_errors:rate5m{code="500"} / ignoring(code) method:http_requests:rate5m
```

这将返回一个结果向量，其中包含最近 5 分钟对每种方法的状态请求为 500 的HTTP请求的比例。如果没有`ignoring(code)`，将不会有匹配项，因为数据指标不会共享同一组标签。 带有 method 标签值为`put`和`del`的条目不匹配，所以不会显示在结果中：

```text
{method="get"}  0.04            //  24 / 600
{method="post"} 0.05            //   6 / 120
```

### 多对一和一对多向量匹配 <a id="many-to-one-and-one-to-many-vector-matches"></a>

**多对一**和**一对多**向量匹配指的是"一"侧的每个向量元素可以与"多"侧的多个元素匹配的情况。必须使用`group_left`或`group_right`修饰符明确表示，其中 left/right 确定哪个向量具有更高的基数。

```text
<vector expr> <bin-op> ignoring(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> ignoring(<label list>) group_right(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_right(<label list>) <vector expr>
```

group 分组修饰符提供的 label list 包含了"一"侧的额外标签。标签只能出现在其中一个 list。结果向量的每个时间序列都必须是唯一的。

_分组修饰符只能用于_[_比较_](operators.md#comparison-binary-operators)_和_[_算术_](operators.md#arithmetic-binary-operators)_运算。默认情况下，像`and`, `unless`和`or`之类的操作与右侧向量中的所有可能条目匹配。_

示例查询：

```text
method_code:http_errors:rate5m / ignoring(code) group_left method:http_requests:rate5m
```

在本例中，左向量每个 method 标签包含一个以上的条目。因此，我们使用`group_left`指明这一点。现在右侧的元素与左侧具有相同方法标签的多个元素匹配：

```text
{method="get", code="500"}  0.04            //  24 / 600
{method="get", code="404"}  0.05            //  30 / 600
{method="post", code="500"} 0.05            //   6 / 120
{method="post", code="404"} 0.175           //  21 / 120
```

_多对一和一对多匹配是应该认真考虑的高级用例。通常，正确使用`ignore(<labels>)`可提供所需的结果。_

## 聚合操作 <a id="aggregation-operators"></a>

Prometheus 支持如下内置的聚合运算符，这些运算符可用于聚合单个即时向量的元素，从而产生具有聚合值的较少元素的新向量：

* `sum` \(在指定维度上求和\)
* `max` \(在指定维度上求最大值\)
* `min` \(在指定维度上求最小值\)
* `avg` \(在指定维度上求平均值\)
* `stddev` \(在指定维度上求标准差\)
* `stdvar` \(在指定维度上求方差\)
* `count` \(统计向量元素的个数\)
* `count_values` \(统计具有相同数值的元素数量\)
* `bottomk` \(样本值中最小的 k个值\)
* `topk` \(样本值中最大的 k个值\)
* `quantile` \(在指定维度上统计 φ-quantile 分位数\(0 ≤ φ ≤ 1\)\)

这些运算符可以用于聚合所有标签维度，也可以通过包含`without`或`by`子句来保留不同的维度。 这些子句可以在表达式之前或之后使用。

```text
<aggr-op> [without|by (<label list>)] ([parameter,] <vector expression>)
或
<aggr-op>([parameter,] <vector expression>) [without|by (<label list>)]
```

`label list`是未加引号的标签列表，结尾可能包含逗号。例如`(label1, label2)`和`(label1, label2,)`都是合法的。

`without`从结果向量中删除列出的标签，而所有其它标签均保留在结果向量中。`by`恰好相反，它会删除`by`子句中未列出的标签，即使它们的标签值在向量的所有元素之间都相同。

只有`count_values`, `quantile`, `topk`和`bottomk`才需要`parameter`参数。

`count_values`为每个唯一的样本输出一个时间序列，每个序列都有一个附加标签。该标签的参数由聚合参数指定，标签值为唯一的样本值。每个时间序列的值是样本值出现的次数。

`topk`和`bottomk`与其它聚合器不同之处在于将输入样本的一个子集\(包括原始标签\)返回到结果向量中。`by`和`without`仅用于划分输入向量。

例如：

如果指标`http_requests_total`具有按照`application`, `instance`和`group`标签散开的时间序列，我们可以通过如下方式计算每个`application`和`group`在所有实例中看到的 HTTP 请求总数：

```text
sum without (instance) (http_requests_total)
# 等价于
sum by (application, group) (http_requests_total)
```

如果我们只对在所有应用程序中看到的 HTTP 请求总数感兴趣，可以简单写为：

```text
sum(http_requests_total)
```

要计算运行每个构建版本的二进制文件数量，可以写为：

```text
count_values("version", build_version)
```

为了后去所有实例中最大的 5 个 HTTP 请求技术，我们可以使用：

```text
topk(5, http_requests_total)
```

## 二元运算符优先级 <a id="binary-operator-precedence"></a>

下面的列出了 Prometheus 中二元运算符的优先级，从最高到最低。 1. `^` 2. `*`, `/`, `%` 3. `+`, `-` 4. `==`, `!=`, `<=`, `<`, `>=`, `>` 5. `and`, `unless` 6. `or`

优先级相同的运算符是左关联的。例如，`2 * 3 % 2`等效于`(2 * 3) % 2`。但`^`是右关联的，所以`2 ^ 3 ^ 2`等效于`2 ^ (3 ^ 2)`。

