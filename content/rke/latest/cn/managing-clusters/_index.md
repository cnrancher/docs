---
title: 7 - 节点管理
weight: 7
aliases:
  - /rke/latest/cn/installation/managing-clusters/
---

## 添加/删除节点

RKE支持为worker和controlplane平面添加/删除[节点]({{< baseurl >}}/rke/latest/cn/config-options/nodes/)。要添加其他节点，修改原始`cluster.yml`文件，指定节点运行的角色。要删除节点，修改原始`cluster.yml`文件。在`添加/删除`节点进行更改后，运行`rke up --config cluster.yml`更新集群。

## 添加/删除工作节点

在配置文件中`添加/删除工作节点`后运行`rke up --update-only`，除了工作节点修改之外，其他修改内容将被忽略。

## 删除Kubernetes集群

执行`rke remove`命令删除Kubernetes集群。

> **注意:** 此命令是不可逆的，将破坏Kubernetes集群。

这个命令对`cluster.yml`中的每个节点执行以下操作:

- 删除部署在其上的Kubernetes组件  
  - `etcd`
  - `kube-apiserver`
  - `kube-controller-manager`
  - `kubelet`
  - `kube-proxy`
  - `nginx-proxy`

> **注意:** 不会从节点中删除Pod。如果重新使用该节点，则在创建新的Kubernetes群集时将自动删除pod。

- 清除每台主机中服务留下的目录:
  - /etc/kubernetes/ssl
  - /var/lib/etcd
  - /etc/cni
  - /opt/cni
  - /var/run/calico
