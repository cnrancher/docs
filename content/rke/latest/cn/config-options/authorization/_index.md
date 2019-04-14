---
title: 9 - RBAC授权
weight: 9
---

Kubernetes支持多个[授权模块](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#authorization-modules)。目前，RKE仅支持RBAC模块。

默认情况下，RBAC已启用。如果要关闭RBAC（不建议关闭），请将授权模式设置为`none`。

```yaml
authorization:
    # 设置`mode: none`禁用rbac认证
    mode: rbac
```
