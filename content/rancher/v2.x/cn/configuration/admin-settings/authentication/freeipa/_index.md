---
title: 配置FreeIPA
weight: 7
---

`Rancher v2.0.5`版本支持

## 先决条件

- 你必须配置[FreeIPA服务器](https://www.freeipa.org/)。
- 在FreeIPA中创建具有`read-only`访问权限的服务帐户。当用户使用API​​密钥发出请求时，Rancher使用此帐户验证组成员身份。
- 在配置之前请先熟悉[外部身份验证配置和主要用户的概念](../authentication/#外部身份验证配置和主要用户)。

1. 使用分配了`管理员角色`的本地用户登录Rancher。
2. 从`全局`视图中，选择`安全>认证`。
3. 选择`FreeIPA`。
4. 配置FreeIPA服务参数。

    你可能需要登录到域控制器才能找到表单中请求的信息。

    >**使用TLS?**
    > 如果证书是自签名的，或者不是来自认可的证书颁发机构，请确保提供完整的链。需要该链来验证服务器的证书。\
    > **用户Search Base和组Search Base**
    > 搜索库允许Rancher搜索FreeIPA中的用户和组。这些字段仅适用于搜索库，不适用于搜索过滤器。
    > - 如果你的用户和组位于同一`search base`中，请仅填写用户搜索库。
    > - 如果你的组位于不同的search base中，则可以选择填写组Search Base。此字段专用于搜索组，但不是必需的。

5. 如果你的FreeIPA服务器不是标准AD架构，请完成自定义架构参数配置。否则，请跳过此步骤。

    >**搜索属性** `搜索属性`字段默认使用三个特定值：`uid|sn|givenName`。配置FreeIPA后，当输入用户或组时，Rancher会自动按`用户ID，姓氏或名字匹配字段`在FreeIPA服务器查询，Rancher以输入`用户/组`为前缀进行搜索。\
    >默认字段值`uid|sn|givenName`，通过管道(`|`)来分隔这些字段，但可以将此配置设置为这些字段的子集。
    > - `uid`: User ID
    > - `sn`: Last Name
    > - `givenName`: First Name\
    >使用此搜索属性，Rancher会为用户和组创建搜索过滤器，但你无法在此字段中添加自己的搜索过滤器。

6. 最后，输入你的FreeIPA用户名和密码，以完成最后的FreeIPA身份验证。
