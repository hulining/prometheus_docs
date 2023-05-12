# 模板参考

Prometheus 支持在告警的的注解和标签以及服务的控制台页面中进行模板化。模板有对本地数据库运行查询，遍历数据，使用条件语句，格式化数据等功能。Prometheus 模板语言基于[Go模板系统](https://golang.org/pkg/text/template/)。

## 数据结构 <a href="#data-structures" id="data-structures"></a>

处理时间序列的的主要数据结构是 sample，定义如下：

```go
type sample struct {
    Labels map[string]string
    Value float64
}
```

样本的数据指标名称在`Labels`字典映射的特殊`__name__`标签中进行编码。

`[]sample`表示 sample 的列表

Go 里的`interface{}`类似于 C 语言中的方法指针

## 函数 <a href="#functions" id="functions"></a>

除了 Go 模板提供的[默认功能](https://golang.org/pkg/text/template/#hdr-Functions)外，Prometheus 还提供简化模板中查询结果处理的功能。

如果在管道中使用函数，则管道值将作为最后一个参数传递。

### 查询相关 <a href="#queries" id="queries"></a>

|      名称     |        参数       |    返回值    |                     解析                     |
| :---------: | :-------------: | :-------: | :----------------------------------------: |
|    query    |   query string  | \[]sample |               查询数据库，不知道返回范围向量              |
|    first    |    \[]sample    |   sample  |               相当于`index a 0`               |
|    label    |   label,sample  |   string  |       相当于`index sample.Labels label`       |
|    value    |      sample     |  float64  |              相当于`sample.Value`             |
| sortByLabel | label,\[]sample | \[]sample | 根据给出的 label(标签)对 samples(数据样本)列表进行排序。是稳定排序 |

`first`, `label`和`value`旨在使查询结果易于在管道中使用。

### 数字相关 <a href="#numbers" id="numbers"></a>

|         名称         |   参数   |   返回值  |                                  解析                                  |
| :----------------: | :----: | :----: | :------------------------------------------------------------------: |
|      humanize      | number | string | 使用[数据指标前缀](https://en.wikipedia.org/wiki/Metric\_prefix)将数字转化为更以读的格式 |
|    humanize1024    | number | string |                  类似于`humanize`，但使用 1024 作为基准而不是 1000                 |
|  humanizeDuration  | number | string |                         将持续时间(以秒为单位)转换为更可读的格式                        |
| humanizePercentage | number | string |                               将比值转化为百分数                              |
|  humanizeTimestamp | number | string |                          将 Unix 时间戳转化为易读的格式                          |

Humanizing 相关函数旨在产生合理的输出以供人们使用，并且不能保证在不同 Prometheus 版本之间返回相同的结果。

### 字符相关 <a href="#strings" id="strings"></a>

|      名称      |             参数             |   返回值   |                                                                    解析                                                                   |
| :----------: | :------------------------: | :-----: | :-------------------------------------------------------------------------------------------------------------------------------------: |
|     title    |           string           |  string |                                  [strings.Title](https://golang.org/pkg/strings/#Title)函数，大写每个单词的第一个字符                                  |
|    toUpper   |           string           |  string |                                  [strings.ToUpper](https://golang.org/pkg/strings/#ToUpper)函数，所有字符转化为大写                                 |
|    toLower   |           string           |  string |                                  [strings.ToLower](https://golang.org/pkg/strings/#ToLower)函数，所有字符转化为小写                                 |
|     match    |        pattern, text       | boolean |                             [regexp.MatchString](https://golang.org/pkg/regexp/#MatchString)函数，测试非锚定的正则表达式匹配                            |
| reReplaceAll | pattern, replacement, text |  string |                     [Regexp.ReplaceAllString](https://golang.org/pkg/regexp/#Regexp.ReplaceAllString)函数，正则表达式替换，非锚定                     |
|   graphLink  |            expr            |  string |       返回[表达式浏览器](https://github.com/hulining/prometheus\_docs/blob/release-v2.19/prometheus/configuration/browser.md)中该表达式的图形视图的路径      |
|   tableLink  |            expr            |  string | 返回[表达式浏览器](https://github.com/hulining/prometheus\_docs/blob/release-v2.19/prometheus/configuration/browser.md)中表达式的表格化("Console")视图的路径 |

### 其它 <a href="#others" id="others"></a>

|    名称    |           参数           |           返回值           |                                 解析                                |
| :------: | :--------------------: | :---------------------: | :---------------------------------------------------------------: |
|   args   |     \[]interface{}     | map\[string]interface{} |           这会将对象列表转换为具有键 arg0, arg1 等的映射。旨在允许将多个参数传递给模板。           |
|   tmpl   | string, \[]interface{} |         nothing         | 类似于内置的`template`函数，但允许使用非文字作为模板名称。注意，假定其结果是安全的，并且不会自动转义。仅在控制台中可用。 |
| safeHtml |         string         |          string         |                        将字符串标记为 HTML，不需要自动转义                       |

## 模版类型差异 <a href="#template-type-differences" id="template-type-differences"></a>

每种类型的模板都提供了可用于参数化模板的不同信息，且有一些其它的区别。

### 告警字段模板 <a href="#alert-field-templates" id="alert-field-templates"></a>

`.Value`, `.Labels`和`.ExternalLabels`分别包含告警值，告警标签和全局匹配的扩展标签。为了方便起见，他们也暴露为`$value`, `$labels`和`$externalLabels`变量。

### 控制台模板 <a href="#console-templates" id="console-templates"></a>

控制台暴露在`/consoles/`上，并且模板文件来源于`-web.console.templates`参数指向的目录。

控制台模板使用 [html/template](https://golang.org/pkg/html/template/) 渲染，且提供了自动转义功能。要绕过自动转义，请使用`safe*`函数。

URL 参数可作为`.Params`中字典映射使用。要访问具有相同的名称的多个 URL 参数，`.RawParams`是每个参数的列表值的映射。URL 路径在`.Path`中可用，不包括`/consoles/`前缀。全局配置的外部扩展标签可作为`.ExternalLabels`使用。对于这四个变量，还有便捷变量：`$rawParams`，`$params`, `$path`和`$externalLabels`。

控制台还可以访问在`-web.console.libraries`参数指向的目录中的`*.lib`文件中找到的`{{define"templateName"}}...{{end}}`定义的所有模板。由于这是一个共享的空间，请注意避免与其他用户发生冲突。以`prom`，`_prom`和`__`开头的模板名称保留供 Prometheus 使用，上面列出的函数也是如此。
