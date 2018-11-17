---
title: 1 - Helm HA安装
weight: 1
---

对于生产环境，我们建议使用高可用安装Rancher，以便保证用户可以始终访问Rancher Server。当Rancher安装在Kubernetes集群中时，Rancher将使用集群的`etcd`存储数据，并利用Kubernetes调度实现高可用性。

> **重要:** 为了获得最佳性能，我们建议这个Kubernetes集群只用于Rancher Server。

## 推荐的架构

- Rancher的DNS应解析为第4层负载均衡器
- Load Balancer应将端口`TCP/80`和`TCP/443`转发到Kubernetes集群中的所有节点IP上。
- Ingress控制器将`HTTP`重定向到`HTTPS`, 并将作为端口TCP/443上的SSL/TLS终止。
- Ingress控制器将流量转发到Rancher pod的TCP/80端口。

<sup>HA Rancher安装了第4层负载均衡器，描述了入口控制器的SSL终止 </sup>
![Rancher HA]({{< baseurl >}}/img/rancher/ha/rancher2ha.svg)

## 必备工具

此架构安装需要以下CLI工具。请确保您的这些工具已安装并配置了`PATH`环境变量。

- [kubectl]({{< baseurl >}}/rancher/v2.x/cn/installation/download/#kubectl) - Kubernetes命令行工具。
- [rke]({{< baseurl >}}/rancher/v2.x/cn/installation/download/#rancher-rke) - Rancher Kubernetes Engine用于构建Kubernetes集群。
- [helm]({{< baseurl >}}rancher/v2.x/cn/installation/download/#helm) - Kubernetes的包管理。

## 附加安装选项

- [从HA RKE Add-on安装迁移]({{< baseurl >}}/rancher/v2.x/cn/upgrades/migrating-from-rke-add-on/)

## 以前的安装方法

> **重要提示: RKE add-on安装方法仅支持Rancher v2.0.8**
>
>如果您目前正在使用`RKE add-on`安装的集群，请参阅[从HA RKE Add-on安装迁移]({{< baseurl >}}/rancher/v2.x/cn/upgrades/migrating-from-rke-add-on/)进行升级迁移。

- [RKE add-on安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install/rke-ha-install/)

## [下一步：创建四层负载均衡](./create-nodes-lb/)