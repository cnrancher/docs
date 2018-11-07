---
title: 4 - RKE离线升级
weight: 4
---

>此方法仅适用于Rancher:v2.0.8及之前的版本

要离线升级Rancher Server，需要先把最新稳定版本的 `Rancher Server`镜像以及其他系统组件镜像同步到私有镜像仓库，然后运行upgrade命令。

## 离线升级Rancher Server

  1. 按照[离线安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/air-gap-installation/)需求，拉取最新稳定版本镜像并同步到私有镜像仓库；

  2. 按照[单节点安装升级]({{< baseurl >}}/rancher/v2.x/cn/upgrades/single-node-upgrade/)的操作步骤完成升级

      >**注意:** 在单节点升级时，请在运行docker run命令时将私有镜像仓库地址添加到镜像名中。
      >
      >例如: `docker run -d <registry.yourdomain.com:port>/rancher/rancher:latest`
