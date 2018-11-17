---
title: 2 - Rancher HA安装
weight: 2
---

## Rancher HA安装方法选择

因为版本升级架构改变，Rancher 2.x目前有两种`HA`安装方法：

- [RKE HA](./helm-rancher)

    这种HA安装方法基于RKE部署K8S集群和Rancher HA，此方法支持`2.0.8`以及之前的版本。

- [Helm HA](./rke-ha-install)

    这种方法通过RKE部署K8S集群，再通过Helm去安装Rancher HA。此方法支持`2.0.8`以后的版本。