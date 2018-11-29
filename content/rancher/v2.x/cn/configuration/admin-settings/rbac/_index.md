---
title: 3 - 用户角色
weight: 3

---

在Rancher中，每个人都以`用户`身份进行身份验证这是一个允许您访问Rancher的登录信息。如[Authentication]({{< baseurl >}}/rancher/v2.x/cn/admin-settings/authentication/)中所述，用户可以是本地的也可以是外部的。

配置外部身份验证后，`用户`页面上显示的用户将更改。

- 如果以本地用户身份登录，则只显示本地用户。
- 如果以外部用户登陆，则会显示外部用户和本地用户。

## 用户和角色

一旦用户登录到Rancher，他们的`_authorization_`或他们在系统中的访问权限由_global permissions_和_cluster和project roles_决定。

- [全局权限]({{< baseurl >}}/rancher/v2.x/cn/admin-settings/rbac/global-permissions/):

    在特定集群的范围之外定义用户授权。

- [集群和项目角色]({{< baseurl >}}/rancher/v2.x/cn/admin-settings/rbac/cluster-project-roles/):

    在为其分配角色的特定集群或项目中定义用户授权。

全局权限以及集群和项目角色都在[Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)之上实现。