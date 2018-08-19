---
title: 3 - 端口需求
weight: 3
---

要保证Rancher正常运行，需要主机或者安全策略打开以下端口。使用云服务创建集群(如Amazon EC2或DigitalOcean),Rancher会自动打开这些端口。下图显示了Rancher的基本端口要求。如果需要了解更多，请查阅下表。

![Basic port Requirements]({{< baseurl >}}/img/rancher/port-communications.png)

## Rancher nodes

运行 `rancher/rancher` 容器的主机

### Rancher nodes-入站规则

| 协议 | 端口 | Source                                                       | Description                                          |
| -------- | ---- | ------------------------------------------------------------ | ---------------------------------------------------- |
| TCP      | 80   | Load balancer/proxy that does external SSL termination       | Rancher UI/API when external SSL termination is used |
| TCP      | 443  | etcd nodes <br />controlplane nodes<br />worker nodes<br />Hosted/Imported Kubernetes<br />any that needs to be able to use UI/API | Rancher agent, Rancher UI/API, kubectl               |

### Rancher nodes-出站规则

| 协议 | 端口 | Destination                                                  | Description                                   |
| -------- | ---- | ------------------------------------------------------------ | --------------------------------------------- |
| TCP      | 22   | Any node IP from a node created using Node Driver            | SSH provisioning of nodes using Node Driver   |
| TCP      | 443  | 35.160.43.145/32  <br />35.167.242.46/32 <br />52.33.59.17/32 | git.rancher.io (catalogs)                     |
| TCP      | 2376 | Any node IP from a node created using Node Driver            | Docker daemon TLS port used by Docker Machine |
| TCP      | 6443 | Hosted/Imported Kubernetes API                               | Kubernetes apiserver                          |

## etcd nodes

具有**etcd**角色的节点

### etcd nodes-入站规则

| 协议 | 端口  | Source                                                       | Description                                                  |
| -------- | ----- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| TCP      | 2376  | Rancher nodes                                                | Docker daemon TLS port used by Docker Machine (only needed when using Node Driver/Templates) |
| TCP      | 2379  | etcd nodes<br />controlplane nodes                           | etcd client requests                                         |
| TCP      | 2380  | etcd nodes<br />controlplane nodes                           | etcd peer communication                                      |
| UDP      | 8472  | etcd nodes<br />controlplane nodes<br />worker nodes         | Canal/Flannel VXLAN overlay networking                       |
| TCP      | 9099  | etcd node itself (local traffic, not across nodes)See [Local node traffic](https://www.cnrancher.com/docs/rancher/v2.x/cn/installation/references/#有关本地节点流量信息) | Canal/Flannel livenessProbe/readinessProbe                   |
| TCP      | 10250 | controlplane nodes                                           | kubelet                                                      |

### etcd nodes-出站规则

| 协议 | 端口 | Destination                                                  | Description                                |
| -------- | ---- | ------------------------------------------------------------ | ------------------------------------------ |
| TCP      | 443  | Rancher nodes                                                | Rancher agent                              |
| TCP      | 2379 | etcd nodes                                                   | etcd client requests                       |
| TCP      | 2380 | etcd nodes                                                   | etcd peer communication                    |
| TCP      | 6443 | controlplane nodes                                           | Kubernetes apiserver                       |
| UDP      | 8472 | etcd nodes<br />controlplane nodes<br />worker nodes         | Canal/Flannel VXLAN overlay networking     |
| TCP      | 9099 | etcd node itself (local traffic, not across nodes)See [Local node traffic](https://www.cnrancher.com/docs/rancher/v2.x/cn/installation/references/#有关本地节点流量信息) | Canal/Flannel livenessProbe/readinessProbe |

## controlplane nodes:

具有**controlplane**角色的主机

### controlplane nodes-入站规则

| 协议 | 端口        | Source                                                       | Description                                                  |
| -------- | ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| TCP      | 80          | Any that consumes Ingress services                           | Ingress controller (HTTP)                                    |
| TCP      | 443         | Any that consumes Ingress services                           | Ingress controller (HTTPS)                                   |
| TCP      | 2376        | Rancher nodes                                                | Docker daemon TLS port used by Docker Machine (only needed when using Node Driver/Templates) |
| TCP      | 6443        | etcd nodes<br />controlplane nodes<br />worker nodes         | Kubernetes apiserver                                         |
| UDP      | 8472        | etcd nodes<br />controlplane nodes<br />worker nodes         | Canal/Flannel VXLAN overlay networking                       |
| TCP      | 9099        | controlplane node itself (local traffic, not across nodes)See [Local node traffic](https://www.cnrancher.com/docs/rancher/v2.x/cn/installation/references/#有关本地节点流量信息) | Canal/Flannel livenessProbe/readinessProbe                   |
| TCP      | 10250       | controlplane nodes                                           | kubelet                                                      |
| TCP      | 10254       | controlplane node itself (local traffic, not across nodes)See [Local node traffic](https://www.cnrancher.com/docs/rancher/v2.x/cn/installation/references/#有关本地节点流量信息) | Ingress controller livenessProbe/readinessProbe              |
| TCP/UDP  | 30000-32767 | Any source that consumes NodePort services                   | NodePort port range                                          |

### controlplane nodes-出站规则

| 协议 | 端口  | Destination                                                  | Description                                     |
| -------- | ----- | ------------------------------------------------------------ | ----------------------------------------------- |
| TCP      | 443   | Rancher nodes                                                | Rancher agent                                   |
| TCP      | 2379  | etcd nodes                                                   | etcd client requests                            |
| TCP      | 2380  | etcd nodes                                                   | etcd peer communication                         |
| UDP      | 8472  | etcd nodes<br />controlplane node<br />sworker nodes        | Canal/Flannel VXLAN overlay networking          |
| TCP      | 9099  | controlplane node itself (local traffic, not across nodes)See [Local node traffic](https://www.cnrancher.com/docs/rancher/v2.x/cn/installation/references/#有关本地节点流量信息) | Canal/Flannel livenessProbe/readinessProbe      |
| TCP      | 10250 | etcd nodescontrolplane nodesworker nodes                     | kubelet                                         |
| TCP      | 10254 | controlplane node itself (local traffic, not across nodes)See [Local node traffic](https://www.cnrancher.com/docs/rancher/v2.x/cn/installation/references/#有关本地节点流量信息) | Ingress controller livenessProbe/readinessProbe |

## worker nodes

具有**worker**角色的主机

### worker nodes-入站规则

| 协议 | 端口        | Source                                                       | Description                                                  |
| -------- | ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| TCP      | 80          | Any that consumes Ingress services                           | Ingress controller (HTTP)                                    |
| TCP      | 443         | Any that consumes Ingress services                           | Ingress controller (HTTPS)                                   |
| TCP      | 2376        | Rancher nodes                                                | Docker daemon TLS port used by Docker Machine (only needed when using Node Driver/Templates) |
| UDP      | 8472        | etcd nodes<br />controlplane node<br />sworker nodes         | Canal/Flannel VXLAN overlay networking                       |
| TCP      | 9099        | worker node itself (local traffic, not across nodes)See [Local node traffic](https://www.cnrancher.com/docs/rancher/v2.x/cn/installation/references/#有关本地节点流量信息) | Canal/Flannel livenessProbe/readinessProbe                   |
| TCP      | 10250       | controlplane nodes                                           | kubelet                                                      |
| TCP      | 10254       | worker node itself (local traffic, not across nodes)See [Local node traffic](https://www.cnrancher.com/docs/rancher/v2.x/cn/installation/references/#有关本地节点流量信息) | Ingress controller livenessProbe/readinessProbe              |
| TCP/UDP  | 30000-32767 | Any source that consumes NodePort services                   | NodePort port range                                          |

### worker nodes-出站规则

| 协议 | 端口  | Destination                                                  | Description                                     |
| -------- | ----- | ------------------------------------------------------------ | ----------------------------------------------- |
| TCP      | 443   | Rancher nodes                                                | Rancher agent                                   |
| TCP      | 6443  | controlplane nodes                                           | Kubernetes apiserver                            |
| UDP      | 8472  | etcd nodes<br />controlplane nodes<br />worker nodes         | Canal/Flannel VXLAN overlay networking          |
| TCP      | 9099  | worker node itself (local traffic, not across nodes)See [Local node traffic](https://www.cnrancher.com/docs/rancher/v2.x/cn/installation/references/#有关本地节点流量信息) | Canal/Flannel livenessProbe/readinessProbe      |
| TCP      | 10254 | worker node itself (local traffic, not across nodes)See [Local node traffic](https://www.cnrancher.com/docs/rancher/v2.x/cn/installation/references/#有关本地节点流量信息) | Ingress controller livenessProbe/readinessProbe |

## 有关本地节点流量信息

Kubernetes healthchecks (`livenessProbe` and `readinessProbe`) are executed on the host itself. On most nodes, this is allowed by default. When you have applied strict host firewall (i.e. `iptables`) policies on the node, or when you are using nodes that have multiple interfaces (multihomed), this traffic gets blocked. In this case, you have to explicitely allow this traffic in your host firewall, or in case of public/private cloud hosted machines (i.e. AWS or OpenStack), in your security group configuration. Keep in mind that when using a security group as Source or Destination in your security group, that this only applies to the private interface of the nodes/instances.

## 使用节点驱动程序时的Amazon EC2安全组

If you are [Creating an Amazon EC2 Cluster](https://rancher.com/rancher/v2.x/en/cluster-provisioning/rke-clusters/node-pools/ec2/), you can choose to let Rancher create a Security Group called `rancher-nodes`. The following rules are automatically added to this Security Group.

**安全组: rancher-nodes**

### 入站规则 

| Type            | 协议 | 端口 Range  | Source                 |
| --------------- | -------- | ----------- | ---------------------- |
| SSH             | TCP      | 22          | 0.0.0.0/0              |
| HTTP            | TCP      | 80          | 0.0.0.0/0              |
| Custom TCP Rule | TCP      | 443         | 0.0.0.0/0              |
| Custom TCP Rule | TCP      | 2376        | 0.0.0.0/0              |
| Custom TCP Rule | TCP      | 2379-2380   | sg-xxx (rancher-nodes) |
| Custom UDP Rule | UDP      | 4789        | sg-xxx (rancher-nodes) |
| Custom TCP Rule | TCP      | 6443        | 0.0.0.0/0              |
| Custom UDP Rule | UDP      | 8472        | sg-xxx (rancher-nodes) |
| Custom TCP Rule | TCP      | 10250-10252 | sg-xxx (rancher-nodes) |
| Custom TCP Rule | TCP      | 10256       | sg-xxx (rancher-nodes) |
| Custom TCP Rule | TCP      | 30000-32767 | 0.0.0.0/0              |
| Custom UDP Rule | UDP      | 30000-32767 | 0.0.0.0/0              |

### 出站规则 

| Type        | 协议 | 端口 Range | Destination |
| ----------- | -------- | ---------- | ----------- |
| All traffic | All      | All        | 0.0.0.0/0   |