---
title: 2 - HA 离线安装
weight: 2
---

## 先决条件

Rancher依靠私有镜像仓库进行离线安装，您必须拥有自己的私有镜像仓库或其他方式将所有Docker镜像分发到每个节点，私有仓库安装请参考 [harbor镜像仓库]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/registry/)

HA安装需要以下CLI工具, 确保这些工具安装在您的工作站或者笔记本上，并且您的工作站或者笔记本需要有权限访问Rancher环境.

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl) - Kubernetes命令行工具。
- [rke](https://rancher.com/docs/rke/latest/en/installation/) - Rancher Kubernetes Engine，用于构建Kubernetes集群的cli。
- [helm](https://docs.helm.sh/using_helm/#installing-helm) - Kubernetes的包管理。
