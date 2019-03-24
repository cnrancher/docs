---
title: 4 - Rancher HA安装
weight: 4
---

> **注意** Rancher HA安装属于高级安装，部署前先了解以下基本技能: \
> 1. 了解`域名与DNS解析`基本概念 \
> 2. 了解`http七层代理和tcp四层代理`基本概念 \
> 3. 了解`反向代理`的基本原理 \
> 4. 了解`SSL证书与域名`的关系 \
> 5. 了解`Helm`的安装使用 \
> 6. 了解`kubectl`的使用 \
> 7. 熟悉`Linux基本操作命令` \
> 8. 熟悉`Docker基本操作命令` 

## Rancher HA安装方法选择

因为版本升级架构改变，Rancher 2.x目前有两种`HA`安装方法：

- [RKE HA](./helm-rancher)

    这种HA安装方法基于RKE部署K8S集群和Rancher HA，此方法支持`2.0.8`以及之前的版本。

- [Helm HA](./rke-ha-install)

    这种方法通过RKE部署K8S集群，再通过Helm去安装Rancher HA。此方法支持`2.0.8`以后的版本。