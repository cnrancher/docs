---
title: 配置OpenLDAP认证
weight: 4
---

`Rancher v2.0.5`版本支持

如果你的组织使用LDAP进行用户身份验证，则可以将Rancher与OpenLDAP服务集成，以提供统一的用户身份验证。

## 一、OpenLDAP身份验证流程

1. 当用户尝试使用LDAP账号登录Rancher时，Rancher使用具有`搜索目录和读取用户/组权限`的服务帐户创建对LDAP服务器的初始绑定。(账号初始化)
2. 然后，Rancher使用基于提供的用户名和配置的属性映射的搜索过滤器在目录中搜索用户。(搜索用户)
3. 找到用户后，使用用户的DN和提供的密码对另一个LDAP绑定请求进行身份验证。()
4. 验证成功后，Rancher将从用户对象的成员资格属性中解析组成员资格，并根据配置的用户映射属性执行组搜索。(搜索组)

> **注意**
> 在配置之前请先熟悉[外部身份验证配置和主要用户的概念](../authentication/#外部身份验证配置和主要用户)。

## 二、先决条件

必须使用LDAP绑定帐户(也称为服务帐户)配置Rancher，以搜索和检索用户和组相关的LDAP条目。建议不要使用管理员帐户或个人帐户，而是在OpenLDAP中创建一个专用帐户，对配置的搜索路径下的用户和组只具有只读访问权限(见下文)。

> **使用TLS？**
> 如果OpenLDAP服务器使用的是自签名证书，或不是来自权威的证书颁发机构，请确保有PEM格式的CA证书(与所有的中间证书连接)。你必须在配置期间设置证书，以便Rancher能够验证证书链。

## 三、配置步骤

## 打开OpenLDAP配置页面

1. 使用系统默认的`admin`帐户登录Rancher UI。
2. 从`全局`视图中，导航到`安全 > 认证`页面
3. 选择OpenLDAP，将显示`配置OpenLDAP服务器`表单。

## 配置OpenLDAP服务器设置

在标题为`1. Configure an OpenLDAP server`的部分中，填写特定于LDAP服务器的信息字段。有关每个参数所需值的详细信息，请参阅下表。

> **注意**
> If you are in doubt about the correct values to enter in the user/group Search Base configuration fields, consult your LDAP administrator or refer to the section [Identify Search Base and Schema using ldapsearch]({{< baseurl >}}/rancher/v2.x/en/admin-settings/authentication/ad/#annex-identify-search-base-and-schema-using-ldapsearch) in the Active Directory authentication documentation.

**Table 1: OpenLDAP server parameters**

| Parameter | Description |
|:--|:--|
| Hostname | Specify the hostname or IP address of the OpenLDAP server |
| Port | Specify the port at which the OpenLDAP server is listening for connections. Unencrypted LDAP normally uses the standard port of 389, while LDAPS uses port 636.|
| TLS | Check this box to enable LDAP over SSL/TLS (commonly known as LDAPS). You will also need to paste in the CA certificate if the server uses a self-signed/enterprise-signed certificate. |
| Server Connection Timeout | 	The duration in number of seconds that Rancher waits before considering the server unreachable. |
| Service Account Distinguished Name | Enter the Distinguished Name (DN) of the user that should be used to bind, search and retrieve LDAP entries. (see [Prerequisites](#prerequisites)). |
| Service Account Password | The password for the service account.  |
| User Search Base | Enter the Distinguished Name of the node in your directory tree from which to start searching for user objects. All users must be descendents of this base DN. For example: "ou=people,dc=acme,dc=com".|
| Group Search Base | If your groups live under a different node than the one configured under `User Search Base` you will need to provide the Distinguished Name here. Otherwise leave this field empty. For example: "ou=groups,dc=acme,dc=com".|

---

### Configure User/Group Schema

If your OpenLDAP directory deviates from the standard OpenLDAP schema, you must complete the **Customize Schema** section to match it.
Note that the attribute mappings configured in this section are used by Rancher to construct search filters and resolve group membership. It is therefore always recommended to verify that the configuration here matches the schema used in your OpenLDAP.

> **Note:**
>
> If you are unfamiliar with the user/group schema used in the OpenLDAP server, consult your LDAP administrator or refer to the section [Identify Search Base and Schema using ldapsearch]({{< baseurl >}}/rancher/v2.x/en/admin-settings/authentication/ad/#annex-identify-search-base-and-schema-using-ldapsearch) in the Active Directory authentication documentation.

#### User Schema

The table below details the parameters for the user schema configuration.

**Table 2: User schema configuration parameters**

| Parameter | Description |
|:--|:--|
| Object Class | The name of the object class used for user objects in your domain. |
| Username Attribute | The user attribute whose value is suitable as a display name. |
| Login Attribute | The attribute whose value matches the username part of credentials entered by your users when logging in to Rancher. This is typically `uid`. |
| User Member Attribute | The user attribute containing the Distinguished Name of groups a user is member of. Usually this is one of `memberOf` or `isMemberOf`. |
| Search Attribute | When a user enters text to add users or groups in the UI, Rancher queries the LDAP server and attempts to match users by the attributes provided in this setting. Multiple attributes can be specified by separating them with the pipe ("\|") symbol. |
| User Enabled Attribute | If the schema of your OpenLDAP server supports a user attribute whose value can be evaluated to determine if the account is disabled or locked, enter the name of that attribute. The default OpenLDAP schema does not support this and the field should usually be left empty. |
| Disabled Status Bitmask | This is the value for a disabled/locked user account. The parameter is ignored if `User Enabled Attribute` is empty. |

---

#### Group Schema

The table below details the parameters for the group schema configuration.

**Table 3: Group schema configuration parameters**

| Parameter | Description |
|:--|:--|
| Object Class | The name of the object class used for group entries in your domain. |
| Name Attribute | The group attribute whose value is suitable for a display name. |
| Group Member User Attribute | The name of the **user attribute** whose format matches the group members in the `Group Member Mapping Attribute`. |
| Group Member Mapping Attribute | The name of the group attribute containing the members of a group. |
| Search Attribute | Attribute used to construct search filters when adding groups to clusters or projects in the UI. See description of user schema `Search Attribute`. |
| Group DN Attribute | The name of the group attribute whose format matches the values in the user's group membership attribute. See  `User Member Attribute`. |
| Nested Group Membership | This settings defines whether Rancher should resolve nested group memberships. Use only if your organisation makes use of these nested memberships (ie. you have groups that contain other groups as members). |

---

### Test Authentication

Once you have completed the configuration, proceed by testing  the connection to the OpenLDAP server. Authentication with OpenLDAP will be enabled implicitly if the test is successful.

> **Note:**
>
> The OpenLDAP user pertaining to the credentials entered in this step will be mapped to the local principal account and assigned admin privileges in Rancher. You should therefore make a conscious decision on which LDAP account you use to perform this step.

1. Enter the **username** and **password** for the OpenLDAP account that should be mapped to the local principal account.
2. Click **Authenticate With OpenLDAP** to test the OpenLDAP connection and finalise the setup.

**Result:**

- OpenLDAP authentication is configured.
- The LDAP user pertaining to the entered credentials is mapped to the local principal (administrative) account.

> **Note:**
>
> You will still be able to login using the locally configured `admin` account and password in case of a disruption of LDAP services.

## Annex: Troubleshooting

If you are experiencing issues while testing the connection to the OpenLDAP server, first double-check the credentials entered for the service account as well as the search base configuration. You may also inspect the Rancher logs to help pinpointing the problem cause. Debug logs may contain more detailed information about the error. Please refer to [How can I enable debug logging]({{< baseurl >}}/rancher/v2.x/en/faq/technical/#how-can-i-enable-debug-logging) in this documentation.
