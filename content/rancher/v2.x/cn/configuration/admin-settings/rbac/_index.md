---
title: 角色
weight: 2

---

Within Rancher, each person authenticates as a _user_, which is a login that grants you access to Rancher. As mentioned in [Authentication]({{< baseurl >}}/rancher/v2.x/en/admin-settings/authentication/), users can either be local or external.

After you configure external authentication, the users that display on the **Users** page changes.

- 如果以本地用户身份登录，则只显示本地用户。
- 如果以外部用户登陆，则会显示外部用户和本地用户。

## 用户和角色

一旦用户登录到Rancher，他们的`_authorization_`或他们在系统中的访问权限由_global permissions_和_cluster和project roles_决定。

- [Global Permissions]({{< baseurl >}}/rancher/v2.x/cn/admin-settings/rbac/global-permissions/):

    Define user authorization outside the scope of any particular cluster.

- [Cluster and Project Roles]({{< baseurl >}}/rancher/v2.x/cn/admin-settings/rbac/cluster-project-roles/):

    Define user authorization inside the specific cluster or project where they are assigned the role.

Both global permissions and cluster and project roles are implemented on top of [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/). Therefore, enforcement of permissions and roles is performed by Kubernetes.
