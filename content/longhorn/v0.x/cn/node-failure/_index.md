---
title: 节点故障处理
---

## Kubernetes节点故障时会发生什么

当`Kubernetes`节点因安装CSI驱动程序而失败时（以下所有内容均基于Kubernetes v1.12并使用默认设置）

1. `一分钟`后，`kubectl --kubeconfig=kube_configxxx.yml  get  nodes`将报告`NotReady`故障节点。
2. 大约`五分钟`后，`NotReady`节点上所有pod的状态将更改为`Unknown或NodeLost`。
3. 如果使用StatefulSet或Deployment进行部署，则需要确定是否可以安全地删除丢失节点上运行的工作负载的pod。具体信息查看[force-delete-stateful-set-pod](https://kubernetes.io/docs/tasks/run-application/force-delete-stateful-set-pod/)

	1. StatefulSet具有稳定的标识，因此Kubernetes不会为用户删除Pod。
	1. Deployment没有稳定的身份，但Longhorn是一种Read-Write-Once类型的存储，这意味着它只能连接到一个Pod。因此，Kubernetes创建的新Pod将无法启动，因为Longhorn卷仍然连接到丢失节点上的旧Pod。
	1. 如果您决定手动（并强制）删除Pod，Kubernetes将花费大约`六分钟`来删除与Pod关联的VolumeAttachment对象，从而最终从丢失的节点分离Longhorn卷并允许它被新的Pod。
