---
title: 2 - RKE HA安装
weight: 2
---

## 重要提示:

RKE HA安装仅支持Rancher v2.0.8以及之前的版本，Rancher v2.0.8之后的版本使用[helm安装Rancher]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/)。

> **注意** Rancher HA安装属于高级安装，部署前先了解以下基本技能: \
> 1. 了解`域名与DNS解析`基本概念 \
> 2. 了解`http七层代理和tcp四层代理`基本概念 \
> 3. 了解`反向代理`的基本原理 \
> 4. 了解`SSL证书与域名`的关系 \
> 5. 了解`Helm`的安装使用 \
> 6. 了解`kubectl`的使用 \
> 7. 熟悉`Linux基本操作命令` \
> 8. 熟悉`Docker基本操作命令`
