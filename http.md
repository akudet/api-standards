<h1 id="http-methods-headers-status">HTTP方法，消息头和状态</h1>

<h2 id="processing">业务处理</h2>

<h3 id="business-capabilities">业务能力和资源建模</h3>

组织的各种业务功能通过API作为一组资源提供给外部访问。功能不能在不同API中重复; 然而资源（例如，用户帐户，信用卡等）可在资源中重复使用。

<h3 id="data-model">数据模型</h3>

用来表示的数据模型 `必须` 符合[RFC 7159](https://tools.ietf.org/html/rfc7159) JSON数据交换格式

<h3 id="serialization">序列化</h3>

* 资源接口 `必须` 支持`application/json` 类型
* 如果传了`Accept` 但是不能返回 `application/json` , `必须` 返回 406 Not Acceptable` 错误

<h3 id="strictness">输入输出严谨性</h3>

APIs `必须`  严格遵守其生产的信息, 并且 `应该` 严格遵守消费的信息


<h2 id="data-resources">HTTP方法</h2>

大多数服务都很容易进入标准数据资源模型，其中主要操作可以用首字母缩写词CRUD（创建，读取，更新和删除）来表示。这些映射非常适合标准HTTP方法。

|HTTP 方法|描述|
|---|---|
| `GET`| 用来 _获取_ 资源 |
| `POST`| 用来 _创建_ 资源，或者 _执行_ 资源上的复杂操作 |
| `PUT`| 用来 _更新_ 资源 |
| `DELETE`| 用来 _删除_ 资源 |
| `PATCH`| 用来 _部分更新_ 资源 |

调用的实际操作必须匹配上表中定义的HTTP方法语义。

* **`GET`** 方法 `禁止` 有副作用。 `禁止` 改变底层资源的状态。
* **`POST`** 方法 `应该` 用于在集合中创建新资源。
    * 示例: 添加信息中添加信用卡, `POST https://api.foo.com/v1/vault/credit-cards`
    * 幂等性: 如果是执行相同的调用(通过使用 [`Foo-Request-Id`](#http-custom-headers) 消息头)并且这个资源已经创建, 那么这个操作 `应该` 是幂等的。
* **`POST`** 方法 `应该` 用于创建新的子资源并建立与主资源的关系。
    * 示例: 要退还事务ID为12345的付款: `POST https://api.foo.com/v1/payments/payments/12345/refund`
* **`POST`** 方法 `可以`用于复杂的操作，以及操作的名称。这也称为 [_controller pattern_](patterns.md#controller-resource) 被认为是RESTful模型的一个例外。它更适用于资源代表业务流程的情况，而操作是作为其一部分执行的步骤或操作。
* **`PUT`** 方法 `应该` 用于更新资源属性或建立从资源到现有子资源的关系; 它通过引用子资源来更新主资源。


<h2 id="http-headers">HTTP 消息头</h2>

HTTP 消息头的目的是用一种统一的独立的方式提供关于消息头和发送者的元数据信息。 HTTP消息头不区分大小写。

* HTTP 消息头 `应该` 只用于cross-cutting concern
* API 实现 `应该不` 依赖于消息头
* 消息头 `禁止` 包含和 API 或者域相关数据
* 如果有可用的HTTP消息头， `必须` 使用这个消息头而不是自定义消息头

<h3 id="assumptions">假设</h3>

**服务消费者和服务提供商:**

* `应该不` 期望某个消息头可用，中间组件可能会丢弃消息头，所以业务逻辑不应该基于消息头。
* `应该不` 假设消息头在HTTP传输中没有改变

**基础架构组件** (Web服务框架，客户机调用库，企业服务总线（ESB），负载均衡器（LB）等):

* `可以` 根据服务状态，请求合法性，直接返回响应，而不需转发此消息。如用户认证或者授权失败时
* `可以` 添加，删除，修改HTTP消息头的值

<h3 id="http-standard-headers">HTTP 标准消息头</h3>

| HTTP Header Name | Description |
|-------------|------------|
| `Accept` | 	此请求标头指定API客户端能够在响应中处理的媒体类型。<br/> 发出HTTP请求的系统 `应该` 发送这个头。<br/> 处理请求的系统 `应该不` 假设它可用。<br/> API 应该支持  `application/json`. |
| `Accept-Charset` | 此请求标头指定API客户端在响应中能够处理的字符集。 <br/> `Accept-Charset` `应该` 包含 `utf-8`.</li></ul> |
| `Content-Language` | 此请求/响应标头用于指定内容的语言。默认为 `zh-CN`. API 客户端 `应该` 通过此消息头来标识数据语言，API `必须` 提供此消息头。|
| `Content-Type` | 此请求/响应标头指示请求或响应主体的媒体类型。  <br/><ul><li>客户端请求如果包含消息体，`必须` 提供此消息头</li><li>服务端响应如果包含消息体，`必须` 提供此消息头</li><li>如果消息体为文本，如JSON，必须包含 `character-set` 参数， 而且必须为 `utf-8`</li><li>当前支持的类型为`application/json`.</li></ul>例子:<pre>(in HTTP request)    Accept: application/json<br/>                     Accept-Charset: utf-8<br/>(in HTTP response)   Content-Type: application/json; charset=utf-8</pre> |



<h3 id="http-custom-headers">HTTP 自定义消息头</h3>

| HTTP Header Name | Description |
|:-------------|------------|
| `Foo-Request-Id`    | API消费者 `可以` 选择使用该消息头发送一个唯一标识请求ID用来跟踪。. <br/><br/>这种消息头也可以用于日志和追踪。 |



<h3 id="http-header-propagation">HTTP 消息头传输</h3>

接收请求头的服务在发送请求或消息时 `必须` 传输相关的自定义消息头和标准消息头。


<h2 id="http-status-codes">HTTP 状态码</h2>

RESTful服务使用HTTP状态代码来指定HTTP方法执行的结果。HTTP协议按范围对状态代码进行分类。

<h3 id="status-code-ranges">状态码范围</h3>

响应API请求时，`必须` 使用以下状态码范围。

|Range|Meaning|
|---|---|
|`2xx`|	成功执行。方法执行有可能以多种方式成功。此状态代码指定成功的方式。.|
|`4xx`| 通常这些是请求，请求中的数据，无效的身份验证或授权等问题。在大多数情况下，客户端可以修改其请求并重新提交。|
| `5xx`| 服务器错误：由于站点中断或软件缺陷，服务器无法执行该方法。5xx范围状态代码不应该用于验证或逻辑错误的处理。 |

<h3 id="status-reporting">状态报告</h3>

成功和失败适用于整个操作，

* `必须` 在`2xx`范围内报告成功， 仅在整个代码执行路径成功时才能返回`2xx` 
* `必须` 在`4xx`或`5xx`范围内报告失败。对于系统错误和应用程序错误都是如此。
* 错误消息 `必须` 和 [`error.json`][22] 定义一致，更多信息参照[Error Handling](#error-handling) 部分
* 返回`4xx`或`5xx`时，`必须`返回根据`error.json`定义的消息体
* 返回`2xx`时 `禁止` 返回根据`error.json`定义的消息体，或者任何类型的错误码，作为响应的一部分
* 对于`4xx`范围的客户都错误，错误消息 `应该 `为客户端提供足够的信息，以确定导致错误的原因以及如何解决错误
* 对于`5xx`范围的服务端错误，错误消息 `应该` 避免向客户端公开服务内部实现，对于外部和内部API都应如此。服务端开发人员，应根据日志和跟踪工具来获取相关信息。

<h3 id="allowed-status-codes">允许的状态码列表</h3>

所有REST API `必须` 仅使用以下状态代码。`加粗`的状态码为API开发人员 `应该`使用的状态码。其余的主要用于Web服务框架开发人员报告与安全性，内容协商等相关的框架级错误。

* API `禁止` 返回此表中未定义的状态码。
* API `可以` 返回此表中定义的一些状态码。

| Status Code | Description |
|-------------|-------------|
| **`200 OK`** | 通用成功执行。 |
| **`201 Created`** | 用作对`POST`方法执行的响应，以指示成功创建资源。如果资源已经创建（例如，通过先前执行相同的方法），则服务器应返回状态代码`200 OK`。 |
| **`202 Accepted`** | 用于异步方法执行以指定服务器已接受请求并将在以后执行它。有关更多详细信息，请参阅 [Asynchronous Operations](patterns.md#asynchronous-operations). |
| **`204 No Content`** | 服务器已成功执行该方法，但没有实体主体要返回。|
| **`400 Bad Request`** | 服务器无法理解该请求。使用此状态代码指定<br/> <ul><li>作为有效负载一部分的数据无法转换为基础数据类型。</li><li>数据不是预期的数据格式。</li><li>必填字段不可用。</li><li>简单数据验证类型的错误。</li></ul>|
| `401 Unauthorized` | 该请求需要身份验证，但是没有提供。注意这和`403 Forbidden`之间的区别. |
| **`403 Forbidden`** | 虽然客户端可能具有有效凭据，但未授权客户端访问该资源。如果业务级别授权失败，API也可以使用此代码。例如，Accound持有人没有足够的资金。|
| **`404 Not Found`** | 服务器未找到与请求URI匹配的任何内容。这意味着URI不正确或资源不可用。例如，可能是该密钥在数据库中不存在数据。|
| `405 Method Not Allowed` | 服务器尚未实现请求的HTTP方法。这通常是API框架的默认行为。|
| `406 Not Acceptable` | 当服务器无法使用客户端请求的媒体类型返回响应的有效负载时，它必须返回此状态码。例如，如果客户端发送`Accept: application/xml` 消息头, 但是API 只能生成`application/json`, 服务器`必须` 返回`406`. |
| `415 Unsupported Media Type` | 当无法处理请求的有效负载的媒体类型时，服务器必须返回此状态代码。例如，如果客户端发送`Content-Type: application/xml` 消息头, 但是API 只能处理`application/json`, 服务器 `必须` 返回`415`. |
| **`422 Unprocessable Entity`** | 无法执行请求的操作，可能需要与当前请求之外的API或进程进行交互。这与`500`响应不同，因为没有系统性问题限制API执行请求。|
| `429 Too Many Requests` | 果用户，应用程序或令牌的速率限制已超过预定义值，则服务器必须返回此状态码。附加[RFC 6585](https://tools.ietf.org/html/rfc6585)定义状态码 |
| **`500 Internal Server Error`** | 这可能是系统错误或应用程序错误，通常表示尽管客户端似乎提供了正确的请求，但服务器上出现意外情况。.`500` 响应指示服务器端软件缺陷或站点停运 `500` `应该不` 作为客户端验证和代码逻辑错误处理 |
| `503 Service Unavailable` | 由于临时维护，服务器无法处理服务请求。|

<h3 id="mapping">HTTP方法与状态码映射</h3>

对于每个HTTP方法，API开发人员 `应该` 只使用此表中标记为“X”的状态代码。

| Status Code | 200 Success | 201 Created |202 Accepted | 204 No Content | 400 Bad Request |  404 Not Found |422 Unprocessable Entity | 500 Internal Server Error |
|-------------|:------------|:------------|:------------|:---------------|:----------------|:---------------|:------------------------|:--------------------------|
| `GET`       | X         |               |               |                  | X             | X              | **`X`**           | X                       |
| `POST`      | X         | X         | **`X`**           |                 |  X             | **`X`**              | **`X`**               | X                       |
| `PUT`       | X             |               | **`X`**           | X           | X             | X              | **`X`**           | X                       |
| `PATCH`     | X             |               |               | X           | X            | X              | **`X`**           | X                       |
| `DELETE`    | X             |               |               | X           | X             | X              | **`X`**               | X                       |

* `GET`: `GET` 主要目的是检索资源，成功时返回 `200` 消息头返回资源内容。 资源集合为空时，`200`, 任然是正确的。如果资源项被软删除，但是接口不应该暴露删除状态时，返回`404`更恰当..
* `POST`: `POST` 主要目的是创建资源， 如果资源不存在并且作为操作的一部分创建了，那么 `应该` 返回`201`
    * 在成功执行时，在响应主体中返回对所创建的资源的引用（以链接或资源标识符的形式）。
    * 幂等语义：如果这是后续执行相同的调用(通过[`Foo-Request-Id`](#http-custom-headers) 消息头) 并且资源已经创建，那么 `应该` 返回`200`状态码。
	* 如果使用子资源（'控制器'或数据资源），并且主资源标识符不存在，`404`是应使用的状态码。
* `POST` 在作为[controller pattern](patterns.md##controller-resource), 使用时, `200` 是应使用的状态码。
* `PUT`: `PUT` 方法 `应该` 返回 `204`状态码， 因为在大多数情况，更新资源成功时没有必要返回任何内容。
    * 在极少数情况下，客户端可能需要服务端生成的值，为了优化流程（客户端在`PUT`请求后执行`GET`)。
    在这种情况下，返回`200`和响应体是可以的。
* `PATCH`: 同`PUT`方法遵循相同的语义，`204`无响应体
	* `200`+响应体，应该尽力避免。因为`PATCH`执行部分更新，对同一资源的多次调用很常见，如果返回，可能导致大量的带宽占用。
* `DELETE`: `DELETE` 方法 `应该` 返回 `204` 状态码。
    * `DELETE` `MUST` 具有幂等性，即使资源已经删除任然应该返回 `204`， 一般客户端并不关心资源是在这次操作中删除的还是在之前的操作删除的。因此应该返回`204`而不是`404`
