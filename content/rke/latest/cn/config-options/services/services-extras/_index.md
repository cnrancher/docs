---
title: Extra Args, Extra Binds, and Extra Environment Variables
weight: 2
---

## 扩展参数（Extra Args）

所有的Kubernetes服务，都可以通过`extra_args`参数更新现有值。

`v0.1.3`之后，使用`extra_args`将添加新参数并覆盖现有默认值。`v0.1.3`之前，使用`extra_args`只会添加新参数，无法更改。

```yaml
services:
    kube-controller:
      extra_args:
        cluster-name: "mycluster"
```

### 扩展挂载（Extra Binds）

可以使用`extra_binds`参数向服务添加额外的卷绑定。

```yaml
services:
    kubelet:
      extra_binds:
        - "/host/dev:/dev"
        - "/usr/libexec/kubernetes/kubelet-plugins:/usr/libexec/kubernetes/kubelet-plugins:z"
```

### 扩展环境变量（ Extra Environment Variables）

通过使用`extra_env`参数，可以将其他环境变量添加到服务中。

```yaml
services:
    kubelet:
      extra_env:
        - "HTTP_PROXY=http://your_proxy"
```
