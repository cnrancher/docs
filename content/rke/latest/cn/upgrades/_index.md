---
title: 3 - 升级
weight: 3
---

## 版本升级

RKE通过更改[系统镜像]({{< baseurl >}}/rke/latest/cn/config-options/system-images//)的镜像版本来支持版本升级。

例如，要改变已部署Kubernetes版本，只需在部署Kubernetes集群的cluster.yml中，修改`rancher/hyperkube`标签从`v1.9.7到v1.10.3`,

原YAML

```bash
system-images:
    kubernetes: rancher/hyperkube:v1.9.7
```

更新后YAML

```bash
system-images:
    kubernetes: rancherhyperkube:v1.10.3
```

在`cluster.yml`配置文件更新后，执行`rke up`升级Kubernetes。

```bash
rke up --config cluster.yml
```

首先，RKE将使用本地的`kube_config_cluster.yml`，在升级到最新的镜像之前，确认Kubernetes集群中现有组件的版本。

> **注意:** RKE不支持回滚到以前的版本。

## 服务升级

可以通过更改`Services参数或extra_args`并使用更新的配置文件重新运行rke up来[升级服务]({{< baseurl >}}/rke/latest/cn/config-options/services/)。

> **注意:** `service_cluster_ip_range`或者`cluster_cidr`不能更改，因为对这些参数的任何更改都将导致集群损坏。目前，网络容器不会自动升级。

## 附加组件升级

从v0.1.8开始，支持附加组件升级。
 