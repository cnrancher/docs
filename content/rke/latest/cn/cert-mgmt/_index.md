---
title: 6 - 证书管理
weight: 6
---

> 从v0.2.0版本开始可用

证书是Kubernetes集群的重要组成部分，作用于所有Kubernetes集群组件。RKE通过`RKE cert `命令来处理证书。

## 生成证书签名请求(CSRs)和密钥

如果希望使用权威CA机构颁发的证书，可以使用RKE[生成证书签名请求(CSRs)文件和密钥]({{< baseurl >}}/rke/v0.1.x/en/installation/certs/#generating-certificate-signing-requests-csrs-and-keys).

您可以把`证书签名请求(CSRs)文件和密钥`交给权威CA机构进行签名颁发证书。在证书签名之后，RKE可以通过[自定义证书]({{< baseurl >}}/rke/v0.1.x/en/installation/certs/) 功能来使用这些证书。

## 证书轮换

默认情况下，Kubernetes集群需要证书，RKE将自动为集群生成证书。在证书过期之前以及证书受到破坏时，轮换些证书非常重要。

证书轮换之后，Kubernetes组件将自动重新启动。证书轮换可用于下列服务:

- etcd
- kubelet
- kube-apiserver
- kube-proxy
- kube-scheduler
- kube-controller-manager

RKE可以通过一些简单的命令轮换自动生成的证书:

* 使用相同的CA轮换所有服务证书
* 使用相同的CA为单个服务轮换证书
* 轮换CA和所有服务证书

当您准备轮换证书时, RKE 配置文件 `cluster.yml`是必须的。运行`rke cert rotate`命令时，可通过`--config`指定配置路径。

### 使用相同的CA轮换所有服务证书

使用 `rke cert rotate`进行相同的CA轮换所有服务证书

```bash
$ rke cert rotate

INFO[0000] Initiating Kubernetes cluster
INFO[0000] Rotating Kubernetes cluster certificates
INFO[0000] [certificates] Generating Kubernetes API server certificates
INFO[0000] [certificates] Generating Kube Controller certificates
INFO[0000] [certificates] Generating Kube Scheduler certificates
INFO[0001] [certificates] Generating Kube Proxy certificates
INFO[0001] [certificates] Generating Node certificate   
INFO[0001] [certificates] Generating admin certificates and kubeconfig
INFO[0001] [certificates] Generating Kubernetes API server proxy client certificates
INFO[0001] [certificates] Generating etcd-xxxxx certificate and key
INFO[0001] [certificates] Generating etcd-yyyyy certificate and key
INFO[0002] [certificates] Generating etcd-zzzzz certificate and key
INFO[0002] Successfully Deployed state file at [./cluster.rkestate]
INFO[0002] Rebuilding Kubernetes cluster with rotated certificates
.....
INFO[0050] [worker] Successfully restarted Worker Plane..
```

### 使用相同的CA在单个服务上轮换证书

使用`--service` 指定单个服务，比如`kubelet`:

`rke cert rotate --service kubelet`

```bash
$ rke cert rotate --service kubelet
INFO[0000] Initiating Kubernetes cluster
INFO[0000] Rotating Kubernetes cluster certificates
INFO[0000] [certificates] Generating Node certificate
INFO[0000] Successfully Deployed state file at [./cluster.rkestate]
INFO[0000] Rebuilding Kubernetes cluster with rotated certificates
.....
INFO[0033] [worker] Successfully restarted Worker Plane..
```

### 轮换CA和所有服务证书

如果需要轮换CA证书，则需要轮换所有服务证书，因为它们需要使用新轮换的CA证书签名。在CA和所有服务证书轮换之后，这些服务将自动重新启动，以便使用新证书运行。

轮换CA证书将导致重新启动其他系统服务，这些系统服务也将使用新的CA证书。这包括:

- Networking pods (canal, calico, flannel, and weave)
- Ingress Controller pods
- KubeDNS pods

`rke cert rotate --rotate-ca`

```bash
$ rke cert rotate --rotate-ca
INFO[0000] Initiating Kubernetes cluster
INFO[0000] Rotating Kubernetes cluster certificates
INFO[0000] [certificates] Generating CA kubernetes certificates
INFO[0000] [certificates] Generating Kubernetes API server aggregation layer requestheader client CA certificates
INFO[0000] [certificates] Generating Kubernetes API server certificates
INFO[0000] [certificates] Generating Kube Controller certificates
INFO[0000] [certificates] Generating Kube Scheduler certificates
INFO[0000] [certificates] Generating Kube Proxy certificates
INFO[0000] [certificates] Generating Node certificate   
INFO[0001] [certificates] Generating admin certificates and kubeconfig
INFO[0001] [certificates] Generating Kubernetes API server proxy client certificates
INFO[0001] [certificates] Generating etcd-xxxxx certificate and key
INFO[0001] [certificates] Generating etcd-yyyyy certificate and key
INFO[0001] [certificates] Generating etcd-zzzzz certificate and key
INFO[0001] Successfully Deployed state file at [./cluster.rkestate]
INFO[0001] Rebuilding Kubernetes cluster with rotated certificates
```
