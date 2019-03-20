---
title: 2 - 单节点离线升级
weight: 2
---

要离线升级Rancher Server，需要先同步最新的Rancher镜像到私有镜像仓库中，然后再进行下面的升级操作。

## 先决条件

从v2.0.7开始，Rancher引入了`system`项目，该项目是自动创建的，用于存储Kubernetes需要运行的重要命名空间。在升级到`v2.0.7+`前，请检查环境中有没有创建`system`项目，如果有则删除。`并检查确认所有系统命名空间未分配到任何项目下，如果有则移到出去，以防止集群网络问题。`

## 离线升级Rancher Server

1. 按照离线安装方法[准备离线镜像]({{< baseurl >}}/rancher/v2.x/cn/installation/air-gap-installation/prepare-private-reg/)。

2. 按照[单节点升级]({{< baseurl >}}/rancher/v2.x/cn/upgrades/single-node-upgrade/)的方法，进行Rancher server升级。

    >**注意:** 在执行单节点升级时，`docker run`参数中的镜像名，需要添加私有仓库地址。
    >
    > 例如: `<registry.yourdomain.com:port>/rancher/rancher:latest`

>**升级后出现网络问题？**
>
> 请查阅[Restoring Cluster Networking]({{< baseurl >}}/rancher/v2.x/en/upgrades/upgrades/namespace-migration/#restoring-cluster-networking)。
