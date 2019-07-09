<h1 id="error-handling">错误处理</h1>

根据HTTP规范，可以使用整数和消息来指定请求执行的结果。 该号码称为 _状态代码_，消息称为 _原因短语_。原因短语是用于说明响应结果的人类可读消息。HTTP状态代码`4xx`表示客户端错误，`5xx`表示服务器端错误。然而，这些状态代码和读原因短语不足以表达足够的错误信息。

因此，API `必须` 返回根据[`error.json`][22] 定义的错误消息。推荐属于某一名称空间的接口有一对应的 [错误种类](#error-catalog) 于之关联。

<h2 id="error-schema">错误信息结构</h2>

根据`error.json`定义的错误消息 `必须` 包含以下字段。

* `name`: 错误的唯一名称，推荐从错误种类的 [`error_spec.json#name`][24]里获取。
* `message`: 错误的描述信息。此消息 `必须` 是问题的描述，而不是关于如何解决它的建议。推荐从错误种类的[`error_spec.json#message`][24] 里获取。
* `details`: 包含错误实例的数组。前端错误`4xx`时必须字段 。
	* `field`: 如果请求是json则为错误字段的[JSON 指针][23]， 否则为路径参数或者查询参数名称。
	* `value`: 错误字段的值
	* `issue`: 错误原因，因该从错误种类的[`error_spec_issue.json#issue`][25] 获取。
	* `location`: 错误字段出现的位置，应为`query`, `path`, `body` 如果不存在此字段则默认为`body`.
* `debug_id`: 在服务器端生成的唯一错误标识符，并与日志相关联。

<h3 id="input-errors">输入验证错误</h3>

在验证请求时，应按以下顺序解决各种问题：

| 请求验证原因 | HTTP状态码 |
|------------- | -------------|
| JSON 格式错误 | `400 Bad Request`|
| 包含客户端可以更改的验证错误 | `400 Bad Request`|
| 由于请求之外的原因无法执行，请求格式正确，但是由于语义无法执行| `422 Unprocessable Entity`|

<h2 id="error-samples">错误消息示例</h2>

本节提供一些示例用来演示[`error.json`][22] 的使用。

<h4 id="sampleresponse-invalid">验证错误响应 - 单个字段</h4>

下面的例子演示单个字段的类型为`VALIDATION_ERROR`的错误 应该这是个客户端错误所以应该返回`400 Bad Request`

```

{  
   "name":"VALIDATION_ERROR",
   "details":[  
      {  
         "field":"#/credit_card/expire_month",
         "issue":"必要字段缺失",
         "location":"body"
      }
   ],
   "debug_id":"123456789",
   "message":"提供数据无效",
}
```

<h4 id="sampleresponse-multi">验证错误响应 - 多个字段</h4>

下面的例子演示了多个字段的类型为`VALIDATION_ERROR`错误, 注意`details`字段为一个数组，列举了所有该类型的错误。由于都是客户段错误蓑衣应该返回`400 Bad Request`

``` 

{  
   "name": "VALIDATION_ERROR",
   "details": [  
      {  
         "field": "/credit_card/expire_month",
         "issue": "必要字段缺失",
         "location": "body"
      },
      {  
         "field": "/credit_card/currency",
         "value": "XYZ",
         "issue": "货币编码无效",
         "location": "body"
      }
   ],
   "debug_id": "123456789",
   "message": "提供字段无效",
}

```

<h4 id="sampleresponse-422">语义验证错误响应</h4>

在客户端请求格式正确，但是请求接口需要其他接口或者程序进行交互时，应该返回`422 Unprocessable Entity` 

```
{  
   "name": "BALANCE_ERROR",
   "debug_id": "123456789",
   "message": "用户余额不足",
}
```
    
<h2 id="error-catalog">错误种类</h2>

一个错误种类包含一个名称空间的错误定义集合。每个错误定义包含错误名称，错误消息，原因等。

<h3 id="reasons-to-catalog-errors">使用错误种类原因</h3>

1. _外部化_ 错误消息。开发人员擅长写代码并不一定擅长写错误消息。开发人员编写错误消息时一般会对上下文做过多的假设。不方便进行消息的变更，如果错误消息写在代码中，如需要更改提示消息，开发人员需要更改相应代码，并重新部署代码。
2. _区域化_ 错误消息。外部化错误消息后，便于其他团队对不同语言的错误进行编写，而不需要重新部署服务，或者服务开发人员协助。

<h3 id="error-catalog-schema">错误种类结构</h3>

每个错误种类，主要有4种对象。

1. [`error_catalog.json`][26] 
	* `namespace`: API名称空间
	* `language`: 错误种类语言
	* `errors`: 一个或多个错误目录项。
2. [`error_catalog_item.json`][27] 定义错误种类中的项目，目前只包含`error_spec`字段，表示错误定义。
3. [`error_spec.json`][24] 错误定义，包含以下属性。
	* `name`: 错误的唯一名称。
	* `message`:错误的描述信息。此消息 `必须` 是问题的描述。此值 `必须` 设置到 [`error.json#message`][22] 字段
	* `log_level`: 该错误对应的日志级别，`禁止` 在错误响应中返回。
	* `http_status_codes`: 该错误对应的HTTP状态码。
	* `suggested_application_actions`: 建议使用API​​的应用程序开发人员可以采取的操作，以便解决错误情况。
	* `suggested_user_actions`: 建议使用API​​的应用程序的用户可以采取的实际操作，以便解决错误情况。
	* `issues`: 该错误相关的原因， 在`error_spec_issue.json`里定义，每个原因关联. 每个原因关联一个 [`error_details.json`][28]. 
* [`error_spec_issue.json`][25]  定义错误的原因。比如，可能有多种原因触发`400 BAD REQUEST`. 当返回响应时，每个原因必须对应[`error_details.json`][28] 的一个项目。
	* `id`: 每个错误种类唯一的id，用来搜索某一错误种类下的 `error_spec_issue`
	* `issue`: 错误的原因，这个值 `必须` 用来设置[`error_details.json#issue`][28] 字段。 原因字符串可以用变量。

<h3 id="string-format">字符串格式化</h3>

对于用Java实现的API服务，错误种类中的各种字符串 `必须` 使用Java的 `printf-style` 进行格式化。 推荐使用Java的[格式化规范](https://docs.oracle.com/javase/7/docs/api/java/util/Formatter.html#summary) 定义`message` 和 `issue`字段的值。
 
强烈建议服务开发人员使用 [Java String Formatter](https://docs.oracle.com/javase/7/docs/api/java/util/Formatter.html) 等工具来将错误种类的格式化的字符串生成错误消息。

比如`error_spec` 定义了 `message` 字段为 `Could not add card due to failure to comply with guideline %s` 必须使用formatter生成错误消息。

```
com.foo.platform.error.ErrorSpec errorSpec = <从错误种类里获取错误定义>

com.foo.platform.error.Error error = new Error();
error.setName(errorSpec.getName());
error.setDebugId("debugId-777");
error.setLogLevel(errorSpec.getLogLevel());
String errorMessageString = String.format(errorSpec.getMessage(), "GUIDELINE: XYZ");
error.setMessage(errorMessageString);

List<Detail> details = new ArrayList<>();
Detail detail = new Detail();
String issueString = String.format(errorSpec.getIssues().get(0).getIssue(), ... <variables in issue string>);
detail.setIssue(issueString);
details.add(detail);
error.setDetails(details);

Response response = Response.status(Response.Status.BAD_REQUEST).entity(error).encoding(MediaType.APPLICATION_JSON).build();
throw new WebApplicationException(response);
```


<h3 id="samples">示例</h3>

本节提供一些错误种类的示例。

#### 示例种类: 名称空间 : payments

```
{
	"namespace": "payments",
	"language": "zh-CN",
	"errors": [{
		"error_spec": {
			"name": "VALIDATION_ERROR",
			"message": "无效请求，见details",
			"log_level": "ERROR",
			"http_status_codes": [
				400
			],
			"issues": [{
				"id": "InvalidCreditCardType",
				"issue": "数据非法，必须为Visa卡"
			}],
			"suggested_application_actions": [
				"提供一个合适的信用卡类型，重新请求"
			]
		}
	}, {
		"error_spec": {
			"name": "PAYEE_ACCOUNT_LOCKED_OR_CLOSED",
			"message": "收款人账号被锁定或关闭",
			"log_level": "ERROR",
			"http_status_codes": [
				422
			],
			"legacy_code": "PAYER_ACCOUNT_LOCKED_OR_CLOSED",
			"issues": [{
				"id": "PayerAccountLocked",
				"issue": "收起该付款的账号被禁用或者锁定，无法继续交易"
			}],
			"suggested_user_actions": [
				"通过contact@foo.com联系客户服务"
			]
		}
	}]
}
```

#### 示例种类: 名称空间 : wallet

```
{
	"namespace": "wallet",
	"language": "zh-CN",
	"errors": [{
		"error_spec": {
			"name": "INVALID_ISSUER_DETAILS",
			"message": "非法的发证机关详情",
			"log_level": "ERROR",
			"http_status_codes": [
				400
			],
			"issues": [{
				"id": "ISSUER_DATA_NOT_FOUND",
				"issue": "发证机关数据未找到"
			}],
			"suggested_application_actions": [
				"提供发证人相关信息并重新请求"
			]
		}
	}, {
		"error_spec": {
			"name": "INSTRUMENT_BLOCKED",
			"message": "操作被冻结",
			"log_level": "ERROR",
			"http_status_codes": [
				422
			],
			"issues": [{
				"id": "BankAccountBlocked",
				"issue": "银行账户由于多次随机操作已被冻结"
			}],
			"suggested_user_actions": [
				"通过contact@foo.com联系客户服务."
			]
		}
	}]
}
```

####  示例种类: 名称空间 : payment-networks

```
{
	"namespace": "payment-networks",
	"language": "zh-CN",
	"errors": [{
		"error_spec": {
			"name": "VENDOR_TIMEOUT",
			"message": "等待第三方响应时超时",
			"log_level": "ERROR",
			"http_status_codes": [
				504
			],
			"suggested_application_actions": [
				"请稍后重试"
			]
		}
	}, {
		"error_spec": {
			"name": "INTERNAL_TIMEOUT",
			"message": "内部错误，处理请求超时。交易状态未知。",
			"log_level": "ERROR",
			"http_status_codes": [
				500
			],
			"suggested_application_actions": [
				"请联系contact@foo.com并提供相应的debug_id和关联id，进行分析。"
			]
		}
	}]
}
```

[0]: https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm "RESTful Architectural Style"
[1]: https://tools.ietf.org/html/rfc5988#section-4 "Link Relation Type"
[2]: http://tools.ietf.org/html/draft-zyp-json-schema-04 "JSON schema draft-04"
[3]: http://json-schema.org/draft-04/hyper-schema# "JSON hyper-schema draft-04"
[4]: http://json-schema.org/latest/json-schema-hypermedia.html#anchor17 "Link Description Objects (LDO)"
[5]: http://www.iana.org/assignments/link-relations/link-relations.xhtml "IANA's list of standardized link relations"
[6]: https://tools.ietf.org/html/rfc6570 "URI template"
[7]: uri-component-names "URI Definition"
[8]: https://tools.ietf.org/html/rfc3986 "RFC 3968"
[9]: http://tools.ietf.org/html/draft-zyp-json-schema-04 "draft-04"
[10]: http://tools.ietf.org/html/draft-zyp-json-schema-03 "draft-03"
[11]: http://swagger.io/specification "OpenAPI"
[12]: http://swagger.io/specification/#schemaObject "OpenAPI Schema"
[13]: v1/schema/json/draft-04 "Common Types"
[14]: v1/schema/json/draft-04/README.md "README"
[15]: v1/schema/json/draft-04/money.json "money_json"
[16]: https://github.com/googlei18n/libaddressinput/wiki/AddressValidationMetadata "i18n-api"
[17]: https://www.w3.org/TR/html51/sec-forms.html#autofill-field "HTML 5.1 autofill"
[18]: v1/schema/json/README_address.md "README Address"
[19]: v1/schema/json/draft-04/address_portable.json "address_portable.json"
[20]: https://www.informatica.com/content/dam/informatica-com/global/amer/us/collateral/other/addressdoctor-cloud-2_user-guide.pdf "Address Doctor"
[21]: https://www.ietf.org/rfc/rfc3339.txt "RFC3339" 
[22]: v1/schema/json/draft-04/error.json "error.json"
[23]: http://tools.ietf.org/html/rfc6901 "JavaScript Object Notation (JSON) Pointer"
[24]: v1/schema/json/draft-04/error_spec.json "error_spec.json"
[25]: v1/schema/json/draft-04/error_spec_issue.json "error_spec_issue.json"
[26]: v1/schema/json/draft-04/error_catalog.json "error_catalog.json"
[27]: v1/schema/json/draft-04/error_catalog_item.json "error_catalog_item.json"
[28]: v1/schema/json/draft-04/error_details.json "error_details.json"
[29]: http://techbus.safaribooksonline.com/book/web-development/web-services/9780596809140 "RESTful Web Services Cookbook"
[30]: http://json.org/ "JSON"




