---
title: 安装
weight: 3
---

本章节包含在开发和生产环境中安装Rancher的配置说明。该部分还包含配置负载均衡和SSL证书以配合Rancher使用的补充文档。

对于生产环境，我们建议使用高可用安装Rancher，以便保证用户可以始终访问Rancher Server。

当Rancher以，LOCAL方式安装在Kubernetes集群中时，Rancher将复用LOCAL集群的`etcd`进行数据存储，并利用Kubernetes的调度机制实现Rancher自身的高可用性。
