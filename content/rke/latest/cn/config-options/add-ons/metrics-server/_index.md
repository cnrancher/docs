---
title: 4 - Metrics Server
weight: 4
---

默认情况下，RKE将以`Deployment`方式部署[Metrics Server](https://github.com/kubernetes-incubator/metrics-server) 以提供群集中资源的指标。

Metrics Server使用的镜像位于[`system_images`]({{< baseurl >}}/rke/latest/cn/config-options/system-images/)。对于每个Kubernetes版本，都有一个与Metrics服务器关联的默认镜像，但是可以通过更改`system_images`中的镜像版本来覆盖默认镜像。

## 禁用 Metrics Server

_可用版本 v0.2.0_

从 rke 0.2.0开始支持禁用Metrics Server，在rke 配置文件中，设置`monitoring.provider为none`以禁用Metrics Server

```yaml
monitoring:
    provider: none
```
