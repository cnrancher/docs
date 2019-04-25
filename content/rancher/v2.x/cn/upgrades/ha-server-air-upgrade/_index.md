---
title: 4 - RKE HA离线升级
weight: 4
---

>此方法仅适用于Rancher:v2.0.8及之前的版本

## 一、先决条件

从v2.0.7开始，Rancher引入了`system`项目，该项目是自动创建的，用于存储Kubernetes需要运行的重要命名空间。在升级到`v2.0.7+`前，请检查环境中有没有创建`system`项目，如果有则删除。`并检查确认所有系统命名空间未分配到任何项目下，如果有则移到出去，以防止集群网络问题。`

要离线升级Rancher Server，需要先把最新稳定版本的 `Rancher Server`镜像以及其他系统组件镜像同步到私有镜像仓库，然后运行upgrade命令。

## 二、离线升级Rancher Server

  1. 按照[离线安装]({{< baseurl >}}/rancher/v2.x/cn/installation/air-gap-installation/)需求，拉取最新稳定版本镜像并同步到私有镜像仓库;

  2. 按照[RKE HA升级]({{< baseurl >}}/rancher/v2.x/cn/upgrades/ha-server-upgrade/)的操作步骤完成升级;

      >**注意:** 在升级时，需要在镜像名前添加私有仓库地址。比如: `<registry.yourdomain.com:port>/rancher/rancher:stable (或者rancher/rancher:latest)`
