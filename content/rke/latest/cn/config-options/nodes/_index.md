---
title: 1 - 节点
weight: 1
---

该`nodes`参数是`cluster.yml`文件中唯一必需的部分,它为RKE指定集群节点ip地址、ssh凭证以及这些节点将在Kubernetes集群中的角色。

```yaml
nodes:
    nodes:
    - address: 1.1.1.1
      user: ubuntu
      role:
      - controlplane
      - etcd
      ssh_key_path: /home/user/.ssh/id_rsa
      port: 2222
    - address: 2.2.2.2
      user: ubuntu
      role:
      - worker
      ssh_key: |-
        -----BEGIN RSA PRIVATE KEY-----

        -----END RSA PRIVATE KEY-----
    - address: 3.3.3.3
      user: ubuntu
      role:
      - worker
      ssh_key_path: /home/user/.ssh/id_rsa
      ssh_cert_path: /home/user/.ssh/id_rsa-cert.pub
    - address: 4.4.4.4
      user: ubuntu
      role:
      - worker
      ssh_key_path: /home/user/.ssh/id_rsa
      ssh_cert: |-
        ssh-rsa-cert-v01@openssh.com AAAAHHNza...
    - address: example.com
      user: ubuntu
      role:
      - worker
      hostname_override: node3
      internal_address: 192.168.1.6
      labels:
        app: ingress
```

## 一、地址

`address`参数设置节点的主机名或IP地址，RKE必须能够连接到此地址。

## 二、内部地址

在具有多IP的主机上，通过`internal_address`参数指定集群内部通讯地址，如果没有设置`internal_address`，则使用`address`进行主机间通信。

比如同一区域的云主机，拥有互通的vpc网络地址和公网地址，在通过rke创建集群的时候，`address`地址需要填写公网地址，因为rke需要与节点通信，而vpc网络是无法直接访问的。这种场景下，设置`internal_address`为vpc网络地址，这样K8S集群运行在vpc网络上，保证了集群内部的网络性能。如果不设置`internal_address`，将会使用`address`地址组件集群，这样网络性能会很差。

## 三、覆盖主机名

`hostname_override`用于为RKE提供一个友好的名称，以便在Kubernetes中注册节点时使用。这个主机名不需要是一个可路由的地址，但是它必须是一个有效的[Kubernetes资源名](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names)。如果没有设置`hostname_override`，则在Kubernetes中注册节点时使用`address`指令。

> **注意:** 当配置[云提供商]({{< baseurl >}}/rke/latest/cn/config-options/cloud-providers/)时, 可能需要使用`覆盖主机名`才能正确使用云提供商。 [AWS云提供商](https://kubernetes.io/docs/concepts/cluster-administration/cloud-providers/#aws)是个例外，`hostname_override` 字段将被明确忽略。

## 四、SSH Port

指定连接到节点时要使用的节点`port`。默认端口是`22`。

## 五、SSH Users

指定连接到节点时要使用的节点`user`,该用户必须是Docker组成员，或者允许写入节点的Docker套接字。

## 六、SSH Key Path

`ssh_key_path`,指定连接到节点时要使用的SSH私钥路径，每个节点的默认密钥路径是`~/.ssh/id_rsa`。

> **注意:** 如果您具有可在所有节点上使用的私钥，则可以在集群级别设置SSH密钥路径,每个节点中设置的SSH密钥路径优先使用。

## 七、SSH Key

相对`ssh_key_path`，`ssh_key`可以设置实际密钥内容，而不是设置SSH密钥文件的路径。

## 八、SSH Certificate Path

连接到节点时要使用的已签名SSH证书，可以通过`ssh_cert_path`指定ssh_cert路径。

## 九、SSH Certificate

相对`ssh_cert_path`，`ssh_cert`可以指定实际证书内容，而不是设置签名SSH证书的路径。

## 十、Kubernetes Roles

RKE中，支持三种角色设置: `controlplane，etcd和worker`。

- `controlplane`: 运行Kubernetes master相关组件(kube-apiserver、kube-controller-manager、kube-scheduler、kube-proxy、kubelet)；
- `etcd`: 运行etcd服务(etcd)；
- `worker`: 作为Kubernetes woker节点运行(kube-proxy、kubelet、)

节点角色不是互斥的，可以将角色组合分配给节点，也可以通过升级对已有的节点进行角色修改。

> **Note:** 在`v0.1.8`之前，`工作负载/pod`可能在具有`worker或controlplane角色`的节点上运行，但从v0.1.8开始，它们将仅部署到`worker`节点上。

### 1、**etcd**角色

节点设置**etcd**角色，`etcd`容器将在这些节点上运行。Etcd是一个分布式可靠的键值存储，存储所有Kubernetes状态，是集群中最重要的组件，是集群的唯一数据来源，通常需要3个、5个或更多节点来创建HA配置。

具有etcd角色的节点上设置的[污点](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)如下所示:

Taint Key                              | Taint Value  | Taint Effect
---------------------------------------|--------------|--------------
`node-role.kubernetes.io/etcd`         | `true`       | `NoExecute`

### 2、**controlplane**角色

节点配置**controlplane**角色，Kubernetes的无状态组件将在这些节点上运行。

具有**controlplane**角色的节点上设置的污点如下所示：

Taint Key                              | Taint Value  | Taint Effect
---------------------------------------|--------------|--------------
`node-role.kubernetes.io/controlplane` | `true`       | `NoSchedule`

### 3、**worker**角色

**worker**节点也可以理解为计算节点，部署的应用(workloads或者pods)都将在**worker**节点运行。

## 十一、Docker Socket

Docker套接字默认路径为`/var/run/docker.sock`，如果安装的非标准安装docker,则可以通过`docker_socket`设置Docker套接字地址。

## 十二、Labels

可以通过`node_selector`为节点设置标签，标签可用于应用部署调度节点选择。