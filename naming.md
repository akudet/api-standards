<h1 id="naming-conventions">命名规范</h1>

本节描述了URI，查询参数，资源，字段和枚举的命名约定。

<h3 id="uri-naming-convention">URI 命名规范</h3>

以下是API的URI特定命名约定准则的简要说明。该规范使用括号“（）”进行分组，使用星号“*”指定零次或多次出现，使用括号“[]”表示可选字段。

```
[scheme"://"][host[':'port]]"/v" major-version '/'namespace '/'resource ('/'resource)* '?' query
```

* URI `必须` 以字母开头，并且只使用小写字母。
* URI路径中的文字/表达式 `应该` 使用连字符(-)分隔。
* 查询字符串中的文字/表达式 `应该` 使用下划线(_)分隔。
* URI 路径和查询参数 `必须` 使用 `utf-8` 编码
* `应该` 在URI中使用名词复数来识别数据资源的集合。
  * `/invoices`
  * `/statements`
* 单个资源 `可以` 直接存在于集合资源下面。
  * `/invoices/{invoice_id}`
* 集合子资源 `可以` 直接存在于单个资源之下。
  * `/invoices/{invoice_id}/items`
* 单个子资源 `可以` 直接存在于单个资源之下，但更建议使用顶级资源
  * `/invoices/{invoice_id}/items/{item_id}`
  * 建议: `/invoice-items/{invoice_item_id}`
* 资源id应该根据该[建议](#resource-identifiers) 定义

**例子**

* `https://api.foo.com/v1/vault/credit-cards`
* `https://api.foo.com/v1/vault/credit-cards/CARD-7LT50814996943336KESEVWA`
* `https://api.foo.com/v1/payments/billing-agreements/I-V8SSE9WLJGY6/re-activate`

<h3 id="resource-names">资源名称</h3>

在将服务建模为一组资源时，开发人员 `必须` 遵循以下原则：

* `必须` 使用名词，而不是动词。
* `必须` 用名词单数表示单例资源，复数表示集合资源。
    * 用户帐户上自动付款配置的说明
        * `GET /autopay` 返回完整的表示
    *  一系列假设收费：
        * `GET /charges` 返回已经收费的清单
        * `GET /charges/1234` 返回单次充电的完整表示
* `必须` 用小写的资源名称，并且只使用字母数字字符和连字符(-)。

<h3 id="query-parameter-names">查询参数名称</h3>

* `应该` 使用下划线(_)分隔查询字符串中的文字/表达式。
* `必须` 使用百分号编码编码查询参数值。
* `必须` 使用字母，下划线，数字，并且 `应该` 都小写，来表示查询参数名称。
* 查询参数 `应该` 是可选的。

<h2 id="field-names">字段名称</h2>

表示的数据模型必须符合[JSON][30]。值本身可以是对象，字符串，数字，布尔值或对象数组。

* 键名 `必须` 是小写单词，用下划线字符分隔。
    * `foo`
    * `bar_baz`
* 如`is_`或者的前缀`has_` `不应该` 用于布尔类型的键名。
* 表示数组的字段应该使用复数名词命名。

<h2 id="enum-names">枚举名称</h2>

枚举的条目（值）应该只由大写字母数字字符和下划线字符（_）组成。

* `FIELD_10` 
* `NOT_EQUAL`
