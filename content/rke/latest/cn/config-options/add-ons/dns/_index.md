---
title: 3 - DNS Provider
weight: 3
---

RKE提供以下两种DNS Provider：

- CoreDNS
- KUBE-DNS

| RKE版本  | KUBERNETES版本 | 默认DNS Provider |
| :------- | :------------- | :--------------- |
| ≥ v0.2.5 | ≥ v1.14.0      | CoreDNS          |
| ≥ v0.2.5 | ≤ v1.13.x      | KUBE-DNS         |
| ≤ v0.2.4 | 所有版本       | KUBE-DNS         |

使用Kubernetes 1.14及更高版本时，CoreDNS在RKE v0.2.5中成为默认值。如果您使用的RKE版本低于v0.2.5，则默认情况下将部`kube-dns`。

## 一、CoreDNS

*自v0.2.5起可用*

CoreDNS只能在Kubernetes v1.12.0及更高版本上使用。

在RKE v0.2.5中部署Kubernetes 1.14及更高版本时，将默认部署CoreDNS。默认将部署一个POD的Deployment服务，该pod由1个`coredns`容器组成。RKE还将以Deployment方式部署`coredns-autoscaler`，它将根据节点CPU核心和节点的数量来自动扩展coredns pod副本数。有关此逻辑的更多信息，请参见[线性模式](https://github.com/kubernetes-incubator/cluster-proportional-autoscaler#linear-mode)。

用于CoreDNS的镜像在[`system_images`]({{< baseurl >}}/rke/latest/cn/config-options/system-images/)。对于每个Kubernetes版本，都有与CoreDNS关联的默认镜像，但可以通过更改镜像标记来覆盖这些镜像。

### 1. 调度CoreDNS

如果您只想在特定节点上部署CoreDNS pod，则可以在该`dns`部分中设置`node_selector`参数 。在需要运行DNS POD的节点中设置相应的标签。

```yaml
nodes:
    - address: 1.1.1.1
      role: [controlplane,worker,etcd]
      user: root
      labels:
        app: dns

dns:
    provider: coredns
    node_selector:
      app: dns
```

### 2.  配置CoreDNS

- 上游DNS服务器

  默认情况下，CoreDNS将使用主机配置的DNS服务器（`/etc/resolv.conf`）来解析外部域名查询。如果要配置CoreDNS使用的特定上游DNS服务器，可以使用`upstreamnameservers`指令。

  > **注意：** 设置时`upstreamnameservers`，`provider`也需要设置。

  ```yaml
  dns:
      provider: coredns
      upstreamnameservers:
      - 1.1.1.1
      - 8.8.4.4
  ```

## 二、 KUBE-DNS

RKE将部署KUBE-DNS作为与1默认副本数吊舱由3个容器的部署：`kubedns`，`dnsmasq`和`sidecar`。RKE还将部署kube-dns-autoscaler作为部署，它将使用核心和节点的数量来扩展kube-dns部署。有关此逻辑的更多信息，请参见[线性模式](https://github.com/kubernetes-incubator/cluster-proportional-autoscaler#linear-mode)。

用于kube-dns的镜像属于[`system_images`]({{< baseurl >}}/rke/latest/en/config-options/system-images/)。对于每个Kubernetes版本，都有与kube-dns关联的默认镜像，但可以通过更改镜像标记来覆盖这些镜像。

### 1. 调度 kube-dns

_可用版本 v0.2.0_

如果只想在特定节点上部署`kube-dns pod`，可以在`dns`部分设置`node_selector`。`node_selector`中的标签需要与要部署的kube-dns pod的节点上的标签匹配。

```yaml
nodes:
    - address: 1.1.1.1
      role: [controlplane,worker,etcd]
      user: root
      labels:
        app: dns

dns:
    provider: kube-dns
    node_selector:
      app: dns
```

### 2. 配置 kube-dns

- 上游 DNS服务器

  _可用版本 v0.2.0_

  默认情况下，kube-dns将使用已配置的DNS名称服务器(`/etc/resolv.conf`)来执行外部域名查询。如果希望配置特定的上游 DNS服务器供kube-dns使用，可以使用`upstreamnameservers`指令。

  > **注意：** 设置时`upstreamnameservers`，`provider`也需要设置。

  ```yaml
  dns:
      provider: kube-dns
      upstreamnameservers:
      - 1.1.1.1  
      - 8.8.4.4
  ```

### 3. 禁用 kube-dns

_可用版本 v0.2.0_

您可以通过在集群配置中为`DNS.provider设置为none`来禁用默认DNS提供程序。请注意，这将导致您的pod在无法在集群中执行名称解析。

```yaml
dns:
    provider: none
```


