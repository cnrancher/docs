---
title: API
weight: 11
---

## 如何使用API

API具有自己的用户界面，可从Web浏览器访问。这是查看资源、执行操作以及查看等效cURL或HTTP请求和响应的简便方法。要访问它，请单击右上角的用户头像，在API和Keys下，您可以找到URL访问入口以及创建[API keys]({{< baseurl >}}/rancher/v2.x/cn/configuration/user-settings/api-keys/).

## 认证

API请求必须包含身份验证信息，可使用API密钥通过HTTP基本身份验证完成身份验证。[API Keys]({{< baseurl >}}/rancher/v2.x/cn/configuration/user-settings/api-keys/)可以创建新集群、可以访问多个集群/v3/clusters/。API KEY适用于集群和项目角色，限制帐户可以看到的集群和项目以及它们可以采取的操作。

## 请求

API通常是RESTful，但有几个功能可以定义客户端可发现的所有内容，以便可以编写通用客户端，而不必为每种类型的资源编写特定代码。有关通用API规范的详细信息，请[参阅此处](https://github.com/rancher/api-spec/blob/master/specification.md).

- 每种类型都有一个Schema描述:
  - 获取此类型资源集合的URL。
  - 资源可以拥有的每个字段，以及它们的类型，基本验证规则，是否是必需的或可选的等。
  - 可以在此类资源上执行的每个操作，包括其输入和输出（也作为模式）。
  - 允许过滤的每个字段。
  - 哪些HTTP谓词方法可用于集合本身或集合中的单个资源。

- So the theory is that you can load just the list of schemas and know everything about the API.  This is in fact how the UI for the API works, it contains no code specific to Rancher itself.  The URL to get Schemas is sent in every HTTP response as a `X-Api-Schemas` header.  From there you can follow the `collection` link on each schema to know where to list resources, and other `links` inside of the returned resources to get any other information.

- In practice, you will probably just want to construct URL strings.  We highly suggest limiting this to the top-level to list a collection (`/v3/<type>`) or get a specific resource (`/v3/<type>/<id>`).  Anything deeper than that is subject to change in future releases.

- Resources have relationships between each other called links.  Each resource includes a map of `links` with the name of the link and the URL to retrieve that information.  Again you should `GET` the resource and then follow the URL in the `links` map, not construct these strings yourself.

- Most resources have actions, which do something or change the state of the resource.  To use these, send a HTTP `POST` to the URL in the `actions` map for the action you want.  Some actions require input or produce output, see the individual documentation for each type or the schemas for specific information.
- 大多数资源都有操作，这些操作可以做一些事情或改变资源的状态。要使用这些，发送一个HTTP ' POST '到您想要操作的' action '映射中的URL。有些操作需要输入或产生输出，请参阅每种类型的单独文档或模式以获取特定信息。

- 要编辑资源，发送HTTP `PUT`请求到'`links.update`，使用要更改的字段更新资源上的链接。如果链接不可见，那么您没有更新资源的权限。不知道的字段和不可编辑的字段将被忽略。
- 要删除资源，可以向`links.remove`发送HTTP`delete`请求。如果链接不可见，那么您没有删除资源的权限。
- 要创建新的资源，通过HTTP发送`POST`请求到schema中的对应的URL(比如`/v3/<type>`)。

## 过滤

大多数集合可以使用HTTP查询参数在服务端通过公共字段进行过滤。该`filters`显示了可以过滤的字段以及您所做请求的过滤值。API UI具有设置过滤的控件，并向您显示相应的请求。对于简单的“等于”匹配它只是`field=value`。可以将修饰符添加到字段名称中，例如：“字段大于42” `field_gt=42`。有关完整详细信息，请参阅[API规范](https://github.com/rancher/api-spec/blob/master/specification.md#filtering)。

## 排序

大多数集合可以使用HTTP查询参数在公共字段的服务器端进行排序。 “sortLinks”映射显示了哪些排序是可用的，还有URL，可以根据这些URL对集合进行排序。它还包括有关当前响应排序的信息（如果已指定）。

## 分页

默认情况下，API响应的页面分页限制为每页`100`个资源。这可以使用limit查询参数进行更改，最多可以为`1000`，`例如: /v3/pods?limit=1000`。