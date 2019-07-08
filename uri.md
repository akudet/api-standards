<h1 id="uri">URI</h1>

<h2 id="resource-path">Resource Path</h2>

API的资源路径由URI的路径，查询和片段组件组成。它将包括API的主要版本，后跟命名空间，资源名称和可选的一个或多个子资源。

`https://api.foo.com/v1/vault/credit-cards/CARD-7LT50814996943336KESEVWA`

下表列出了上述URI的资源路径的各个部分。

| 路径片段 | 描述 | 定义 |
|---|---|---|
|`v1`| 指定API的主要版本 | API主要版本用于区分同一API的两个向后不兼容的版本。API主要版本是一个整数值，必须作为URI的一部分包含在内。 |
|`vault`| 命名空间 | 命名空间标识符用于提供资源的上下文和范围。它们由API平台实现的业务能力模型中的逻辑边界确定。 |
|`credit-cards`|	资源名称 | 如果资源名称表示资源集合，则资源上的 `GET` 方法应检索资源列表。应使用查询参数来指定搜索条件。 |
|`CARD-7LT50814996943336KESEVWA`| 资源ID | 要从集合中检索特定资源，必须将资源ID指定为URI的一部分。子资源将在下面讨论。 |


<h4 id="subresource-path">子资源</h4>

子资源表示从一个资源到另一个资源的关系。子资源名称提供了关系的含义。

| 例子 | 描述 |
|---|---|
|`GET https://api.foo.com/v1/customer-support/disputes/ABCD1234/documents`| 此调用应返回与争议ABCD1234相关的所有文档。 |
|`GET https://api.foo.com/v1/customer-support/disputes/ABCD1234/documents/102030`| 此调用应仅返回与此争议相关的特定文档的详细信息。 |
|`GET https://api.foo.com/v1/customer-support/disputes/ABCD1234/transactions`| 以下示例应返回与此争议相关的所有事务及其详细信息，因此不应指定特定的事务ID。 |

<h3 id="resource-identifiers">资源标识符</h3>

[资源标识符](#resource-identifier) 标识资源或子资源。`必须` 符合以下准则。

* 资源标识符的生命周期 `必须` 由资源的域模型拥有。
* API `禁止` 使用数据库序列号作为资源标识符。
* 推荐用 [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)， 作为资源标识符。
* 出于安全性和数据完整性的原因，所有子资源ID必须仅限于父资源。<br />
**例子:** `/users/1234/linked-accounts/ABCD`<br /> 即使账号"ABCD"存在，如果它没有链接到用户"1234"，也 `禁止` 返回。
* 枚举值可以作为子资源id
* 禁止 一个资源标识紧跟着另一个资源标识符<br />
**例子:** `https://api.foo.com/v1/payments/payments/12345/102030`
* 资源ID和查询参数值 `必须` 对URI非保留字符以外的任何字符执行URI百分号编码。查询参数值 `必须` 用UTF-8编码。

<h2 id="query-parameters">查询参数</h2>

查询参数是在资源路径之后指定的名称/值对。 在应用以下部分时也应遵循[命名规范](#naming-conventions) 相关内容。

#### 过滤资源集合

* 查询参数 `应该` 仅用于限制资源集合或作为搜索或过滤条件。
* 集合中的资源标识符 `应该不` 用于过滤集合结果，
* 分页参数 `应该` 使用以下语法 `page=1&size=20` 查询第一页，每页20条数据。
* 默认排序顺序 `应该` 被视为未定义和非确定的， 如果需要显式排序顺序，则查询参数`sort` `应该` 与以下一般语法一起使用：`sort={field_name},{asc|desc}&sort={field_name},{asc|desc}`. 例如: `/accounts?sort=date_of_birth,asc&zip_code,desc`，为根据出生日期升序，邮政编码降序查询数据。

#### 单个资源上的查询参数
在使用一个资源（例如 `/v1/payments/billing-plans/P-94458432VR012762KRWBZEUA`）， `应该不` 使用查询参数

#### 为同一查询参数传递多个值

将查询参数用于搜索功能时，通常需要传递多个值。例如，一个资源可能有许多状态，如`OPEN`, `CLOSED`, `INVALID`等。客户端如何查询所有状态为`CLOSED`或`INVALID`的项目。

* `推荐` 通过重复查询参数来实现此功能。
    * 上面的查询可实现为 `?status=CLOSED&status=INVALID`.
    * 查询参数的名称 `应该` 为单数。

* ``推荐不`` 使用逗号分隔的参数值，除非有充分理由。
    * 上面的查询可实现为 `?statuses=CLOSED,INVALID`.
    * 查询参数名称 应该 为复数。
    * API接口必须定义如何对分隔符进行转义。
