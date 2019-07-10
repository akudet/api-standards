<h1 id="api-versioning">API版本控制</h1>

本节介绍如何对API进行版本控制。它描述了API的生命周期状态，版本控制策略，向后兼容性指南，生命周期终止策略等内容。

<h2 id="api-lifecycle">API生命周期</h2>

| 状态 | 描述|
|---------|------------|
|PLANNED | API已安排开发。API发布计划已经建立。 |
|BETA| API运行中，可被选择的新用户在生产环境中作为测试验证和发布新版本API使用。|
|LIVE | API运行中，可被新的用户在生产环境中使用。API版本完全支持。 |
|DEPRECATED | API运行中，可被现在系统中的用户使用。API版本完全支持。BUG会以向后兼容的方式进行修复。API版本对新用户不可用。|
|RETIRED | API已经从生产环境中取消发布，对任何用户都不可用。此状态下的应用必须从生产环境和预发布环境中完全移除。|

<h2 id="api-versioning-policy">API版本控制策略</h2>

API是版本化产品，`必须` 遵循以下版本控制原则。

1. API文档 `必须` 根据以下方式定义版本。 用`v`引入版本， 主要版本必须为序数并且用`1`作为第一个LIVE版本。每个主要版本的次要版本必须为序数，并从`0`开始。
2. 每当API 有增量更新时，不论是否向后兼容，都 `必须` 有一版本号与之对应。以便对接口进行注释审核发布等。
3. API接口地址 `必须` 只反映主要版本号。
4. API版本号策略 `可以` 和服务实现版本不同。
5. 次要要版本号的接口，`必须` 和所有之前的次要版本向后兼容。
6. 主要版本号的接口，`可以` 与之前的主要版本兼容。
	
对应指定的功能集，`必须` 只有一个为 `LIVE` 状态的API版本。保证用户明白应该使用哪个版本的接口，如 v1.2 `RETIRED`, v1.3 `DEPRECATED`, or v2.0 `LIVE`.


<h2 id="backwards-compatibility">向后兼容性</h2>

API `应该` 尽量以向前可扩展的方式设计来管理向后兼容性并且应该避免定义重复的资源和功能和过多的版本标识。

API `必须` 遵循以下原则才能被认为是向后兼容的：

1. 所有改动 `必须` 为增量的。
2. 所有改动 `必须` 为可选的。
3. 语义 `禁止`  改变。
4. 查询参数和请求消息 `必须` 是无序的。
5. 已存在接口的附加功能实现，`必须` 满足下面几点
    1. 一个可选的扩展，或者
    2. 一个对新的子资源的新操作，或者
    3. 如果一个操作无法合理扩展，（比如资源创建），可以通过更改一个已经存在的请求消息但是任然接收所有之前已经存在的参数。

<h3 id="backwards-incompatible-changes">向后兼容须满足的部分条件</h3>

<b>URI的变更</b>

1. `可以` 支持其他不必须的参数。
2. `禁止` 在不使用新的参数时，有任何请求URI行为的改变。
3. `应该不` 添加一个必须的参数。
4. `应该不` 改变已存在的参数，标识，资源的语义。
5. `必须` 识别之前有效的值， 当使用时也 `应该不` 抛出异常。
6. `禁止` 有任何与原URI状态码的改变。
7. `禁止` 有任何HTTP方法的改变。可以 支持新的方法。

<b>JSON对象的变更</b>

1. `必须` 返回与JSON对象中已存在的属性相同的名称和类型。
2. `禁止` 改变为数组的请求参数的类型。
3. `必须` 对整个对象进行
4. `可以` 添加新属性到已有表现中，`应该不` 修改已有属性语义。
5. `禁止` 添加必须的新属性。
6. 基础类型如果在文档中没有说明任何限制条件，客户端 `禁止` 假设其有某种限制。
7. 如果对象的属性在URI中，那么它 `必须` 和URI具有相同的稳定性。
8. `禁止` 有任何对于已存在的枚举值或它的涵义进行变更。


<h2 id="eol-policy">生命周期终止</h2>

生命周期终止（EOL）策略规定了API版本如何从`LIVE` 状态转换为 `RETIRED` 状态。它旨在确保用户从旧API版本迁移到新API版本时有一一致且合理的过渡期，同时有一健康的流程偿还技术债务。

<b>API次要版本EOL</b>

根据版本控制策略，次要API版本必须向后兼容同一主要版本中的先前次要版本。因此在同一主要版本下的次要版本，在新的次要版本变为`LIVE`时立即变为``RETIRED``。因为这些改变并不会影响现存用户，所以没有必要通过`DEPRECATED`状态给客户端进行迁移工作。

<b>API主要版本EOL</b>

根据版本控制策略，主要API版本 `可以` 向后兼容之前的主要版本.因此，在停用主要API版本时，以下规则适用。

1. 主要版本的接口 `禁止` 处于`DEPRECATED`状态， 直到提供所有需保留功能的替换服务为`LIVE`，并且提供明确的迁移方法为止。 这些 `应该` 包括文档和适当的迁移工具和示例代码。
2. API版本 `必须` 处于`DEPRECATED`一定的最少时间，来给用户进行迁移。对于有外部客户的API，`应该` 根据具体情况，可能会需要更长的迁移时间来最小化对业务的影响。
6. 如果处于`LIVE`或 `DEPRECATED`的API版本，如果没有用户，`可以` 立即更改为`RETIRED`状态。

<h3 id="eol-policy-replacement">生命周期终止 - 替换主要版本</h3>

因为新的主要API版本会弃用之前的API版本的方式是一个非常重要的业务决策。API拥有者 `必须` 在进行重大设计和开发工作前论证新版本的合理性。API拥有者 `应该` 从最小化客户影响方面考虑所有其他的可能性。理由应该包括以下内容：

商业案例
1. 新版本提供的客户价值是现有版本无法实现的。
2. 新版本与老版本的成本效益分析。
3. 解释其他解决方案，并解释为何不采用。
4. 如果向后不兼容的改变是为了解决重要的安全问题，1和2条不是必须的。

API设计
1. 新版本所有资源的域模型以及其和老板本的域模型比较。
2. 描述API实现的操作和使用场景。
3. 定义对于老版本相当或者更好的服务级别目标（SLO）。

迁移战略
1. 受影响的现有客户数量; 内部，外部和合作伙伴。
2. 为现有客户提供新版本沟通和支持计划，价值和迁移路径的信息。


<h1 id="deprecation">弃用</h1>

本节描述了随着API的发展而弃用部分API的解决方案。它是API版本控制策略的扩展。


<h3 id="deprecation-terms-used">使用术语</h3>

*`API Element`* 用来代指在API中可以弃用的*部分*。 比如接口地址，查询参数，路径参数，JSON对象的属性，JSON对象，自定义消息头等。

*`old API`* 用来指现在API的主要或次要版本，或者现存的会被你的API替换的API。

*`new API`* 用来指新的API主要或者次要版本，或者拥有取代`old API`的新的不同的API。

*`API definition`* 是以[OpenAPI][11]规范的服务接口定义。接口定义可在`swagger.json`.中找到。

<h2 id="deprecation-requirements">目标</h2>

1. API开发者 `应该` 可以在次要版本中禁用`API Element`。
2. API文档 `必须` 突出显示一个或者多个禁用的元素。
3. API服务端 必须 能在运行时通过请求或者响应通知客户端禁用的元素。以便相关工具能够识别，输出警告和突出显示禁用的元素。
4. 禁用的`API Elements`必须在主要版本的生命周期里继续支持，或者直到没有用户使用（确定这一点的方法由API所有者自行决定，因为它的客户最终会受到影响。

<h2 id="deprecation-solution">解决方案</h2>

以下介绍了如何实现上述目标。该解决方案通过使用注释解决与文档相关的需求，并使用自定义标头来解决与运行时相关的需求。

1. [文档](#deprecation-documentation)
2. [运行](#deprecation-runtime)

<h3 id="deprecation-documentation">文档</h3>

<h4 id="deprecation-annotation">Deprecated注解</h4>

Java`@Deprecated`，可以用来弃用方法，参数，实体类。springfox在生成swagger文档能能够识别这些注解信息，并在文档中标识禁用状态。对于Java语言可以直接使用此注解来对禁用的`API element`进行标识。

<h3 id="deprecation-runtime">运行</h3>

API服务端 `必须` 在请求或者响应中通知客户端禁用的`API element`。

<h4 id="deprecation-header">消息头: Foo-Deprecated</h4>

推荐使用HTTP消息头`Foo-Deprecated`来传达弃用相关信息。服务端在以下情况 必须 传输`Foo-Deprecated`消息头。

1. 调用者在请求中使用了一个或多个不推荐使用的元素。
2. 响应中有一个或多个已弃用的元素。

为了避免弃用相关信息使得响应消息膨胀，推荐API开发人员仅提供一个空的JSON对象作为返回值，如。

```
"Foo-Deprecated": "{}"
```

**注意**: 使用该消息头的用户 `必须` 不依靠消息头的值来进行操作，而`应该` 根据消息头是否存在而进行相应操作。


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




