---
title: 端口需求
weight: 2
---

要保证Rancher正常运行，需要主机或者安全策略打开以下端口。使用云服务创建集群（如Amazon EC2或DigitalOcean），Rancher会自动打开这些端口。  下图显示了Rancher的基本端口要求。如果需要了解更多，请参阅下表。

![Basic port Requirements]({{< baseurl >}}/img/rancher/port-communications.png)

## Rancher nodes

运行 `rancher/rancher` 容器的主机

### Rancher nodes-入站规则

| 协议  | 端口  | 源地址                                                       | 描述                                                 |
| ---- | ---- | :----------------------------------------------------------- | ---------------------------------------------------- |
| TCP  | 80   | Load balancer/proxy that does external SSL termination       | Rancher UI/API when external SSL termination is used |
| TCP  | 443  | etcd nodes  <br/>controlplane nodes  <br />worker nodes  <br />Hosted/Imported Kubernetes  <br />any that needs to be able to use UI/API | Rancher agent, Rancher UI/API, kubectl               |

### Rancher nodes-出站规则

| 协议 | 端口 | 目的地址                                                   | 描述                                        |
| ---- | ---- | ---------------------------------------------------------- | ------------------------------------------- |
| TCP  | 22   | Any node IP from a node created using Node Driver          | SSH provisioning of nodes using Node Driver |
| TCP  | 443  | 35.160.43.145/32<br />35.167.242.46/32<br />52.33.59.17/32 | git.rancher.io (catalogs)                   |
| TCP  | 6443 | Hosted/Imported Kubernetes API                             | Kubernetes apiserver                        |

## etcd nodes

etcd角色的主机

### etcd nodes-入站规则

| 协议 | 端口  | 源地址                                               | 描述                                      |
| ---- | ----- | ---------------------------------------------------- | -------------------------------------- |
| TCP  | 2379  | etcd nodes<br />controlplane nodes                   | etcd client requests                   |
| TCP  | 2380  | etcd nodes<br />controlplane nodes                   | etcd peer 通信端口                       |
| UDP  | 8472  | etcd nodes<br />controlplane nodes<br />worker nodes | Canal/Flannel VXLAN overlay networking |
| TCP  | 10250 | controlplane nodes                                   | kubelet                                |

### etcd nodes-出站规则

| 协议 | 端口 | 目的地址                                               | 描述                                    |
| ---- | ---- | ---------------------------------------------------- | -------------------------------------- |
| TCP  | 443  | Rancher nodes                                        | Rancher agent                          |
| TCP  | 2379 | etcd nodes                                           | etcd client requests                   |
| TCP  | 2380 | etcd nodes                                           | etcd peer communication                |
| TCP  | 6443 | controlplane nodes                                   | Kubernetes apiserver                   |
| UDP  | 8472 | etcd nodes<br />controlplane nodes<br />worker nodes | Canal/Flannel VXLAN overlay networking |

## controlplane nodes

controlplane角色的主机

### controlplane nodes-入站规则

| 协议    | 端口        | 源地址                                                 | 描述                                   |
| ------- | ----------- | ---------------------------------------------------- | -------------------------------------- |
| TCP     | 80          | Any that consumes Ingress services                   | Ingress controller (HTTP)              |
| TCP     | 443         | Any that consumes Ingress services                   | Ingress controller (HTTPS)             |
| TCP     | 6443        | etcd nodes<br />controlplane nodes<br />worker nodes | Kubernetes apiserver                   |
| UDP     | 8472        | etcd nodes<br />controlplane nodes<br />worker nodes | Canal/Flannel VXLAN overlay networking |
| TCP     | 10250       | controlplane nodes                                   | kubelet                                |
| TCP/UDP | 30000-32767 | Any source that consumes NodePort services           | Nodeport 端口范围                      |

### controlplane nodes-出站规则

| 协议 | 端口  | 目的地址                                             | 描述                                   |
| ---- | ----- | ---------------------------------------------------- | -------------------------------------- |
| TCP  | 443   | Rancher nodes                                        | Rancher agent                          |
| TCP  | 2379  | etcd nodes                                           | etcd client requests                   |
| TCP  | 2380  | etcd nodes                                           | etcd peer 通信端口                     |
| UDP  | 8472  | etcd nodes<br />controlplane nodes<br />worker nodes | Canal/Flannel VXLAN overlay networking |
| TCP  | 10250 | etcd nodes<br />controlplane nodes<br />worker nodes | kubelet                                |

## worker nodes

worker角色的主机

### worker nodes-入站规则

| 协议    | 端口        | 源地址                                               | 描述                                   |
| ------- | ----------- | ---------------------------------------------------- | -------------------------------------- |
| TCP     | 80          | Any that consumes Ingress services                   | Ingress controller (HTTP)              |
| TCP     | 443         | Any that consumes Ingress services                   | Ingress controller (HTTPS)             |
| UDP     | 8472        | etcd nodes<br />controlplane node<br />sworker nodes | Canal/Flannel VXLAN overlay networking |
| TCP     | 10250       | controlplane nodes                                   | kubelet                                |
| TCP/UDP | 30000-32767 | Any source that consumes NodePort services           | Nodeport 端口范围                      |

### worker nodes-出站规则

| 协议 | 端口 | 目的地址                                             | 描述                                      |
| ---- | ---- | ---------------------------------------------------- | -------------------------------------- |
| TCP  | 443  | Rancher nodes                                        | Rancher agent                          |
| TCP  | 6443 | controlplane nodes                                   | Kubernetes apiserver                   |
| UDP  | 8472 | etcd nodes<br />controlplane node<br />sworker nodes | Canal/Flannel VXLAN overlay networking |
