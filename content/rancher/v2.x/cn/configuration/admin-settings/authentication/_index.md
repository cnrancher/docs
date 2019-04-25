---
title: 5 - 登录认证
weight: 5
---

Rancher为Kubernetes增强的一个关键功能是集中用户身份验证，此功能允许你的用户使用一组凭据对所有Kubernetes集群进行身份验证。

此集中式用户身份验证是使用Rancher身份验证代理完成的，该代理与Rancher一起安装。此代理会对你的用户进行身份验证，并使用服务帐户将其请求转发给你的Kubernetes集群。

## 外部认证与本地认证

Rancher提供本地身份验证，也可以与外部身份验证服务集成：

| Auth Service                                                                                     | 可用版本  |
| ------------------------------------------------------------------------------------------------ | ---------------- |
| [Microsoft Active Directory]({{< baseurl >}}/rancher/v2.x/en/admin-settings/authentication/ad/)  | v2.0.0           |
| [GitHub]({{< baseurl >}}/rancher/v2.x/en/admin-settings/authentication/github/)                  | v2.0.0           |
| [Microsoft Azure AD]({{< baseurl >}}/rancher/v2.x/en/admin-settings/authentication/azure-ad/)    | v2.0.3           |
| [FreeIPA]({{< baseurl >}}/rancher/v2.x/en/admin-settings/authentication/freeipa/)                | v2.0.5           |
| [OpenLDAP]({{< baseurl >}}/rancher/v2.x/en/admin-settings/authentication/openldap/)              | v2.0.5           |
| [Microsoft AD FS]({{< baseurl >}}/rancher/v2.x/en/admin-settings/authentication/microsoft-adfs/) | v2.0.7           |
| [PingIdentity]({{< baseurl >}}/rancher/v2.x/en/admin-settings/authentication/ping-federate/)     | v2.0.7           |
| [Keycloak]({{< baseurl >}}/rancher/v2.x/en/admin-settings/authentication/keycloak/)              | v2.1.0           |

在大多数情况下，你应该使用外部身份验证服务，因为外部验证服务可以统一的进行用户管理。如果没有外部身份验证服务，你也可以通过本地身份验证来管理Rancher用户。

## 外部身份验证配置和主要用户

外部身份验证的配置需要：

- 一个具有管理员角色的本地管理员账号，例如：`local_admin`
- 一个可以使用外部身份验证服务进行身份验证的外部服务账号，例如：`demo`

  外部身份验证的配置会影响Rancher中本地用户的管理方式。请按照以下列表更好地了解这些效果。

1. 以`本地管理员`登录Rancher，对外部身份验证服务进行完整配置。

    ![Sign In]({{< baseurl >}}/img/rancher/sign-in.png)

2. Rancher将外部服务账号与本地管理员账号联系在一起。这两个账号共享本地管理员用户的用户ID。

    ![Principal ID Sharing]({{< baseurl >}}/img/rancher/principal-ID.png)

3. 完成配置后，Rancher会自动注销本地管理员账号。

    ![Sign Out Local Principal]({{< baseurl >}}/img/rancher/sign-out-local.png)

4. 然后，Rancher将自动以外部服务用户登录。

    ![Sign In External Principal]({{< baseurl >}}/img/rancher/sign-in-external.png)

5. 由于外部服务账号和本地管理员账号共享一个ID，因此在`用户`页面上不会显示外部服务账号的唯一标识。

    ![Sign In External Principal]({{< baseurl >}}/img/rancher/users-page.png)

6. 外部服务账号和本地管理员账号共享相同的访问权限。

> **重要说明** 不管启用哪种外部身份验证服务，Rancher内置的`超级管理员admin`将一直启用。