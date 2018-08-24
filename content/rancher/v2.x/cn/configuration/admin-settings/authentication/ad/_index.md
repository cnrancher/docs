---
title: 配置AD认证
weight: 3
---

如果您的组织使用Microsoft Active Directory作为统一用户管理系统，则Rancher可以集成Active Directory服务以进行统一身份验证。Rancher根据Active Directory管理的用户和组来控制对集群和项目的访问，同时允许最终用户在登录Rancher UI时使用其AD凭据进行身份验证。

Rancher使用LDAP与Active Directory服务通信。因此，Active Directory的身份验证流程与[OpenLDAP](../openldap)身份验证集成方法相同。

> **注意:**
>
> 在开始之前，请熟悉[外部身份验证配置和主要用户](/docs/rancher/v2.x/cn/configuration/admin-settings/authentication/#外部身份验证配置和主要用户)的概念。

## 一、先决条件

您需要通过AD管理员创建或获取新的AD用户，以用作Rancher的`服务帐户`。此用户必须具有足够的权限才能执行LDAP搜索并读取AD域下的用户和组的属性。

通常应该使用域用户帐户(非管理员)来实现此目的，因为默认情况下此用户对域中的大多数对象具有只读权限。

但请注意，在某些锁定的Active Directory配置中，此默认行为可能不适用。在这种情况下，您需要确保服务帐户用户至少具有在基本OU(封闭用户和组)上授予的`读取和列出内容`权限，或者全局授予域。

> **使用TLS？**
>
> 如果AD服务器使用的证书是自签名的，或者不是来自公认的证书颁发机构，请确保手头有PEM格式的CA证书(与所有中间证书连接)。您必须在配置期间设置此证书，以便Rancher能够验证证书。

## 二、配置步骤

### 1、选择Active Directory

1. 使用本地admin帐户登录Rancher UI 。
2. 从全局视图中，导航到`安全>认证`。
3. 选择`Active Directory`，配置AD服务参数。

### 2、配置Active Directory服务

在标题为`1. 配置Active Directory服务器`的部分中，填写特定于Active Directory服务的配置信息。有关每个参数所需值的详细信息，请参阅下表。

> **注意:**
>
> 如果您不确定要在用户/组搜索库字段中输入正确的值, please refer to [Identify Search Base and Schema using ldapsearch](#annex-identify-search-base-and-schema-using-ldapsearch).

**表1：AD服务器参数**

| 参数              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| 主机名            | 指定AD服务器的域名或IP地址                                   |
| 端口              | 指定Active Directory服务器侦听的端口。未加密的LDAP通常使用标准端口`389`，而LDAPS使用端口`636`。 |
| TLS               | 选中此框以启用LDAP over SSL/TLS(通常称为LDAPS)。           |
| 服务器连接超时    | Rancher无法访问AD服务器后等待的持续时间。                    |
| 服务帐户用户名    | 输入对您的登录域具有只读访问权限的AD帐户的用户名(请参阅[先决条件](#一、先决条件))。用户名可以用NetBIOS格式输入(例如: `DOMAIN\serviceaccount`)或UPN格式(例如: `serviceaccount@domain.com`)。 |
| 服务帐户密码      | 服务帐户的密码。                                             |
| 默认登录域        | 使用AD域的NetBIOS名称配置此字段时，在没有域(例如`jdoe`)时输入的用户名将在绑定到AD服务器时自动转换为斜线的NetBIOS登录(例如`LOGIN_DOMAIN\jdoe`)。如果您的用户使用UPN(例如` jdoe@acme.com`)作为用户名进行身份验证，则此字段`必须`为空。 |
| User Search Base  | 目录树中节点的专有名称，从该节点开始搜索用户对象。所有用户必须是此基本DN的后代。例如：`ou=people，dc=acme，dc=com`。 |
| Group Search Base | 如果您的组位于与`User Search Base`配置的节点不同的节点下，则需要在此处提供可分辨名称。否则将其留空。例如：`ou=groups，dc=acme，dc=com`。 |

### 配置用户/组架构

在标题为`2. 自定义架构`的部分中，您必须为Rancher提供与目录中使用的模式相对应的`用户和组`属性的正确配置。

Rancher使用LDAP查询来搜索和检索有关Active Directory中的用户和组的信息，本节中配置的属性映射用于构建搜索过滤器并解析组成员身份。因此，提供的设置反映AD域的实际情况至关重要。

> **注意:**
>
> 如果您不熟悉Active Directory域中使用的架构，请参阅使用ldapsearch识别搜索库和架构以确定正确的配置值。

#### 用户架构

下表详细介绍了用户架构部分配置的参数。

**表2：用户架构配置参数**

| 参数           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| 对象类型        | 域中用户对象的类型名称。                             |
| 用户名属性     | 其值作为显示名称。                             |
| 登录属性       | 该值与用户在登录Rancher时输入的凭据的用户名部分相匹配的属性。如果您的用户使用其UPN(例如“ jdoe@acme.com”)作为用户名进行身份验证，则通常必须将此字段设置为`userPrincipalName`。否则，对于旧的NetBIOS样式的登录名称(例如“jdoe”)通常是这样`sAMAccountName`。 |
| 用户成员属性   | 包含用户所属组的属性。                                       |
| 搜索属性       | 当用户输入文本以在UI中添加用户或组时，Rancher将查询AD服务器并尝试按此设置中提供的属性匹配用户。可以通过使用管道(“\|”)符号分隔多个属性来指定它们。要匹配UPN用户名(例如jdoe@acme.com )，通常应将此字段的值设置为`userPrincipalName`。 |
| 用户启用的属性 | 包含表示用户帐户标志的按位枚举的整数值的属性。Rancher使用它来确定是否禁用了用户帐户。您通常应将此设置保留为AD标准`userAccountControl`。 |
| 禁用状态位掩码 | 这是`User Enabled Attribute`指定已禁用的用户帐户的值。您通常应将此设置保留为Microsoft Active Directory架构中指定的默认值“2”(请参阅[此处](https://docs.microsoft.com/en-us/windows/desktop/adschema/a-useraccountcontrol#remarks))。 |

#### 组架构

下表详细说明了组架构配置的参数。

*表3：组架构配置参数**

| 参数           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| 对象类型         | 域中组对象的类型名称。                               |
| 名称属性       | 其值作为显示名称。                                |
| 组成员用户属性 | 格式与中的组成员匹配的**用户属性**的名称`Group Member Mapping Attribute`。 |
| 组成员映射属性 | 包含组成员的组属性的名称。                                   |
| 搜索属性       | 在将群组添加到集群或项目时用于构建搜索过滤器的属性。请参阅用户架构的说明`Search Attribute`。 |
| 组DN属性       | group属性的名称，其格式与描述用户成员身份的user属性中的值匹配。见 `User Member Attribute`。 |
| 嵌套组成员资格 | 此设置定义Rancher是否应解析嵌套组成员资格。仅在您的组织使用这些嵌套成员资格时使用(即您拥有包含其他组作为成员的组)。 |

### 测试认证

完成配置后，继续测试与AD服务器的连接。如果测试成功，将隐式启用使用已配置的Active Directory进行身份验证。

> **注意:**
>
> 与在此步骤中输入的凭据相关的AD用户将映射到本地主体帐户并在Rancher中分配管理员权限。因此，您应该有意识地决定使用哪个AD帐户执行此步骤。

1. 输入应映射到本地主帐户的AD帐户的**用户名**和**密码**。
2. 单击**使用Active Directory**进行**身份验证**以完成设置。

**Result:**

- Active Directory authentication has been enabled.
- You have been signed into Rancher as administrator using the provided AD credentials.

> **Note:**
>
> You will still be able to login using the locally configured `admin` account and password in case of a disruption of LDAP services.

## Annex: Identify Search Base and Schema using ldapsearch

In order to successfully configure AD authentication it is crucial that you provide the correct configuration pertaining to the hirarchy and schema of your AD server.

The [`ldapsearch`](http://manpages.ubuntu.com/manpages/artful/man1/ldapsearch.1.html) tool allows you to query your AD server to learn about the schema used for user and group objects.

For the purpose of the example commands provided below we will assume:

- The Active Directory server has a hostname of `ad.acme.com`
- The server is listening for unencrypted connections on port `389`
- The Active Directory domain is `acme`
- You have a valid AD account with the username `jdoe` and password `secret`

### Identify Search Base

First we will use `ldapsearch` to identify the Distinguished Name (DN) of the parent node(s) for users and groups:

```
$ ldapsearch -x -D "acme\jdoe" -w "secret" -p 389 \
-h ad.acme.com -b "dc=acme,dc=com" -s sub "sAMAccountName=jdoe"
```

This command performs an LDAP search with the search base set to the domain root (`-b "dc=acme,dc=com"`) and a filter targeting the the user account (`sAMAccountNam=jdoe`), returning the attributes for said user:

![LDAP User]({{< baseurl >}}/img/rancher/ldapsearch-user.png)

Since in this case the user's DN is `CN=John Doe,CN=Users,DC=acme,DC=com` [5], we should configure the **User Search Base** with the parent node DN `CN=Users,DC=acme,DC=com`.

Similarly, based on the DN of the group referenced in the **memberOf** attribute [4], the correct value for the **Group Search Base** would be the parent node of that value, ie. `OU=Groups,DC=acme,DC=com`.

### Identify User Schema

The output of the above `ldapsearch` query also allows to determine the correct values to use in the user schema configuration:

- `Object Class`: **person** [1]
- `Username Attribute`: **name** [2]
- `Login Attribute`: **sAMAccountName** [3]
- `User Member Attribute`: **memberOf** [4]

> **Note:**
>
> If the AD users in our organisation were to authenticate with their UPN (e.g. jdoe@acme.com) instead of the short logon name, then we would have to set the `Login Attribute` to **userPrincipalName** instead.  

We'll also set the `Search Attribute` parameter to **sAMAccountName|name**. That way users can be added to clusters/projects in the Rancher UI either by entering their username or full name.

### Identify Group Schema

Next, we'll query one of the groups associated with this user, in this case `CN=examplegroup,OU=Groups,DC=acme,DC=com`:

```
$ ldapsearch -x -D "acme\jdoe" -w "secret" -p 389 \
-h ad.acme.com -b "ou=groups,dc=acme,dc=com" \
-s sub "CN=examplegroup"
```

This command will inform us on the attributes used for group objects:

![LDAP Group]({{< baseurl >}}/img/rancher/ldapsearch-group.png)

Again, this allows us to determine the correct values to enter in the group schema configuration:

- `Object Class`: **group** [1]
- `Name Attribute`: **name** [2]
- `Group Member Mapping Attribute`: **member** [3]
- `Search Attribute`: **sAMAccountName** [4]

Looking  at the value of the  **member** attribute, we can see that it contains the DN of the referenced user. This  corresponds to the **distinguishedName** attribute in our user object. Accordingly will have to set the value of the `Group Member User Attribute` parameter to this attribute.

In the same way, we can observe that the value in the **memberOf** attribute in the user object corresponds to the **distinguishedName** [5] of the group. We therefore need to set the value for the `Group DN Attribute` parameter to this attribute.

## Annex: Troubleshooting

If you are experiencing issues while testing the connection to the Active Directory server, first double-check the credentials entered for the service account as well as the search base configuration. You may also inspect the Rancher logs to help pinpointing the problem cause. Debug logs may contain more detailed information about the error. Please refer to [How can I enable debug logging]({{< baseurl >}}/rancher/v2.x/en/faq/technical/#how-can-i-enable-debug-logging) in this documentation.
