---
title: 配置AD认证
weight: 3
---

如果你的组织使用Microsoft Active Directory作为统一用户管理系统，则Rancher可以集成Active Directory服务以进行统一身份验证。Rancher根据Active Directory管理的用户和组来控制对集群和项目的访问，同时允许最终用户在登录Rancher UI时使用其AD凭据进行身份验证。

Rancher使用LDAP与Active Directory服务通信。因此，Active Directory的身份验证流程与[OpenLDAP](../openldap)身份验证集成方法相同。

> **注意**
> 在开始之前，请熟悉[外部身份验证配置和主要用户](/docs/rancher/v2.x/cn/configuration/admin-settings/authentication/#外部身份验证配置和主要用户)的概念。

## 一、先决条件

你需要通过AD管理员创建或获取新的AD用户，以用作Rancher的`服务帐户`。此用户必须具有足够的权限才能执行LDAP搜索并读取AD域下的用户和组的属性。

通常应该使用域用户帐户(非管理员)来实现此目的，因为默认情况下此用户对域中的大多数对象具有只读权限。

但请注意，在某些锁定的Active Directory配置中，此默认行为可能不适用。在这种情况下，你需要确保服务帐户用户至少具有在基本OU(封闭用户和组)上授予的`读取和列出内容`权限，或者全局授予域。

> **使用TLS？**
> 如果AD服务器使用的证书是自签名的，或者不是来自公认的证书颁发机构，请确保手头有PEM格式的CA证书(与所有中间证书连接)。你必须在配置期间设置此证书，以便Rancher能够验证证书。

## 二、选择Active Directory

1. 使用本地admin帐户登录Rancher UI 。
2. 从`全局`视图中，导航到`安全>认证`。
3. 选择`Active Directory`，配置AD服务参数。

## 三、配置Active Directory服务

在标题为`1. 配置Active Directory服务器`的部分中，填写特定于Active Directory服务的配置信息。有关每个参数所需值的详细信息，请参阅下表。

> **注意**
> 如果你不确定要在用户/组搜索库字段中输入正确的值, 请查看[使用ldapsearch识别Search Base和架构](#六、附件一：使用ldapsearch识别Search Base和架构).

**表1：AD服务器参数**

| 参数              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| 主机名            | 指定AD服务器的域名或IP地址                                   |
| 端口              | 指定Active Directory服务器侦听的端口。未加密的LDAP通常使用标准端口`389`，而LDAPS使用端口`636`。 |
| TLS               | 选中此框以启用LDAP over SSL/TLS(通常称为LDAPS)。           |
| 服务器连接超时    | Rancher无法访问AD服务器后等待的持续时间。                    |
| 服务帐户用户名    | 输入对你的登录域具有只读访问权限的AD帐户的用户名(请参阅[先决条件](#一、先决条件))。用户名可以用NetBIOS格式输入(例如: `DOMAIN\serviceaccount`)或UPN格式(例如: `serviceaccount@domain.com`)。 |
| 服务帐户密码      | 服务帐户的密码。                                             |
| 默认登录域        | 使用AD域的NetBIOS名称配置此字段时，在没有域(例如`jdoe`)时输入的用户名将在绑定到AD服务器时自动转换为斜线的NetBIOS登录(例如`LOGIN_DOMAIN\jdoe`)。如果你的用户使用UPN(例如` jdoe@acme.com`)作为用户名进行身份验证，则此字段`必须`为空。 |
| User Search Base  | 目录树中节点的专有名称，从该节点开始搜索用户对象。所有用户必须是此基本DN的后代。例如：`ou=people，dc=acme，dc=com`。 |
| Group Search Base | 如果你的组位于与`User Search Base`配置的节点不同的节点下，则需要在此处提供可分辨名称。否则将其留空。例如：`ou=groups，dc=acme，dc=com`。 |

## 四、配置用户/组架构(可选)

> **注意** 如果你的AD服务器为标准配置，那可以跳过此步骤

在标题为`2. 自定义架构`的部分中，你必须为Rancher提供与目录中使用的模式相对应的`用户和组`属性的正确配置。

Rancher使用LDAP查询来搜索和检索有关Active Directory中的用户和组的信息，本节中配置的属性映射用于构建搜索过滤器并解析组成员身份。因此，提供的设置反映AD域的实际情况至关重要。

> **注意**
> 如果你不熟悉Active Directory域中使用的架构，请参阅使用ldapsearch识别搜索库和架构以确定正确的配置值。

### 1、用户架构

下表详细介绍了用户架构部分配置的参数。

**表2：用户架构配置参数**

| 参数           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| 对象类型        | 域中用户对象的类型名称。                             |
| 用户名属性     | 其值作为显示名称。                             |
| 登录属性       | 该值与用户在登录Rancher时输入的凭据的用户名部分相匹配的属性。如果你的用户使用其UPN(例如“ jdoe@acme.com”)作为用户名进行身份验证，则通常必须将此字段设置为`userPrincipalName`。否则，对于旧的NetBIOS样式的登录名称(例如“jdoe”)通常是这样`sAMAccountName`。 |
| 用户成员属性   | 包含用户所属组的属性。                                       |
| 搜索属性       | 当用户输入文本以在UI中添加用户或组时，Rancher将查询AD服务器并尝试按此设置中提供的属性匹配用户。可以通过使用管道(“\|”)符号分隔多个属性来指定它们。要匹配UPN用户名(例如jdoe@acme.com )，通常应将此字段的值设置为`userPrincipalName`。 |
| 用户启用的属性 | 包含表示用户帐户标志的按位枚举的整数值的属性。Rancher使用它来确定是否禁用了用户帐户。你通常应将此设置保留为AD标准`userAccountControl`。 |
| 禁用状态位掩码 | 这是`User Enabled Attribute`指定已禁用的用户帐户的值。你通常应将此设置保留为Microsoft Active Directory架构中指定的默认值“2”(请参阅[此处](https://docs.microsoft.com/en-us/windows/desktop/adschema/a-useraccountcontrol#remarks))。 |

### 2、组架构

下表详细说明了组架构配置的参数。

**表3: 组架构配置参数**

| 参数           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| 对象类型         | 域中组对象的类型名称。                               |
| 名称属性       | 其值作为显示名称。                                |
| 组成员用户属性 | 格式与中的组成员匹配的**用户属性**的名称`Group Member Mapping Attribute`。 |
| 组成员映射属性 | 包含组成员的组属性的名称。                                   |
| 搜索属性       | 在将群组添加到集群或项目时用于构建搜索过滤器的属性。请参阅用户架构的说明`Search Attribute`。 |
| 组DN属性       | group属性的名称，其格式与描述用户成员身份的user属性中的值匹配。见 `User Member Attribute`。 |
| 嵌套组成员资格 | 此设置定义Rancher是否应解析嵌套组成员资格。仅在你的组织使用这些嵌套成员资格时使用(即你拥有包含其他组作为成员的组)。 |

## 五、测试认证

完成配置后，继续测试与AD服务器的连接。如果测试成功，将隐式启用使用已配置的Active Directory进行身份验证。

> **注意**
> 与在此步骤中输入的凭据相关的AD用户将映射到本地主体帐户并在Rancher中分配管理员权限。因此，你应该有意识地决定使用哪个AD帐户执行此步骤。

1. 输入应映射到本地主帐户的AD帐户的**用户名**和**密码**。
2. 单击**使用Active Directory**进行**身份验证**以完成设置。

> **注意**
> 如果LDAP服务中断，你可以使用本地帐户和密码登录。

## 六、附件一：使用ldapsearch识别Search Base和架构

为了成功配置AD身份验证，你必须提供与AD服务器的层次结构和架构相关的正确配置。

该[`ldapsearch`](http://manpages.ubuntu.com/manpages/artful/man1/ldapsearch.1.html)工具可以帮助你查询AD服务器用户和组对象的模式。

处于演示的目的，我们假设:

- Active Directory服务器的主机名为`ad.acme.com`。
- 服务器正在侦听端口上的未加密连接`389`。
- Active Directory域是`acme`
- 拥有一个有效的AD帐户，其中包含用户名`jdoe`和密码`secret`

### 1、识别Search Base

首先，我们将使用`ldapsearch`来识别用户和组的父节点的专有名称(DN)：

```
ldapsearch -x -D "acme\jdoe" -w "secret" -p 389 \
-h ad.acme.com -b "dc=acme,dc=com" -s sub "sAMAccountName=jdoe"
```

此命令执行LDAP搜索，`search base`设置为根域(`-b "dc=acme,dc=com"`)，过滤器以用户(`sAMAccountNam=jdoe`)为目标，返回所述用户的属性：

![LDAP User]({{< baseurl >}}/img/rancher/ldapsearch-user.png)

由于在这种情况下用户的DN是`CN=John Doe,CN=Users,DC=acme,DC=com[5]`，我们应该使用父节点DN配置用户Search Base`CN=Users,DC=acme,DC=com`。

类似地，基于`memberOf`属性[4]中引用的组的DN，组Search Base的正确值应该是该值的父节点，即:`OU=Groups,DC=acme,DC=com`。

### 2、识别用户架构

上述`ldapsearch`查询的输出结果还可以确定用户架构的配置：

- `Object Class`: **person** [1]
- `Username Attribute`: **name** [2]
- `Login Attribute`: **sAMAccountName** [3]
- `User Member Attribute`: **memberOf** [4]

> **注意**
> 如果组织中的AD用户使用他们的UPN(例如`jdoe@acme.com` )而不是短登录名进行身份验证，那么我们必须将`Login Attribute`设置为`userPrincipalName`。\
> 还可以`Search Attribute`参数设置为`sAMAccountName | name`。这样，通过输入用户名或全名，可以通过Rancher UI将用户添加到群集/项目中。

### 3、识别组架构

接下来，我们查询与此用户关联的一个组，在这种情况下`CN=examplegroup,OU=Groups,DC=acme,DC=com`：

```
ldapsearch -x -D "acme\jdoe" -w "secret" -p 389 \
-h ad.acme.com -b "ou=groups,dc=acme,dc=com" \
-s sub "CN=examplegroup"
```

以上命令将显示组对象的属性：

![LDAP Group]({{< baseurl >}}/img/rancher/ldapsearch-group.png)

- `Object Class`: **group** [1]
- `Name Attribute`: **name** [2]
- `Group Member Mapping Attribute`: **member** [3]
- `Search Attribute`: **sAMAccountName** [4]

查看成员属性值，我们可以看到它包含引用用户的DN。这对应用户对象中的`distinguishedName`属性。因此必须将`Group Member User Attribute`参数的值设置为此属性值。

以同样的方式，我们可以观察到用户对象中`memberOf`属性中的值对应于组的`distinguishedName [5]`。因此，我们需要将`Group DN Attribute`参数的值设置为此属性值。

## 七、附件二：故障排除

如果在测试与Active Directory服务器的连接时遇到问题，请检查为服务帐户输入的凭据以及`Search Base`配置。还可以检查Rancher日志以帮助查明问题原因。调试日志可能包含有关错误的更多详细信息，请参考[如何开启debug模式](/docs/rancher/v2.x/cn/faq/technical/#怎么样开启debug模式)。