---
title: 升级
weight: 3
---

## 从Rancher 2.x.x升级

Rancher 2.x.x的每个新版本都支持从上一个进行升级。
根据Rancher安装方案，选择以下升级方案:

- [单节点安装升级](./single-node-upgrade/)
- [HA安装升级](./ha-server-upgrade/)
- [离线升级](./air-gap-upgrade/)

## 从Rancher 1.6.x迁移

在Rancher 2.1发布之前，由于主要的代码重写，所以不支持从Rancher 1.6.x直接升级到Rancher2.x.x。
对于2.1版本，我们计划发布一个将Rancher Compose转换为Kubernetes YAML的工具。这个工具将帮助Cattle用户从Rancher 1.6.x迁移到2.x.x。
在2.1版本发布后至少一年，我们将继续支持Rancher 1.6.x，以便1.6.x用户可以计划并完成迁移。