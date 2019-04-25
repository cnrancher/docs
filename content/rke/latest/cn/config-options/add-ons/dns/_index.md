---
title: 3 - DNS provider
weight: 3
---

默认情况下，RKE 部署[kube-dns](https://github.com/kubernetes/dns)作为您的集群的DNS提供程序。

RKE将部署`kube-dns`作为默认副本数为`1`的`Deployment`。`Pod`由三个容器组成:`kubedns、dnsmasq和sidecar`。RKE还将以`Deployment`方式部署`kube-dn -autoscaler`，它将根据`内核和节点的数量`来扩展`kube-dns`实例数。有关此逻辑的更多信息，请参见[Linear Mode](https://github.com/kubernetes-incubator/cluster-proportional-autoscaler#linear-mode)。

## 一、调度 kube-dns

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

## 二、禁用 kube-dns

_可用版本 v0.2.0_

您可以通过在集群配置中为`DNS.provider设置为none`来禁用默认DNS提供程序。请注意，这将导致您的pod在无法在集群中执行名称解析。

```yaml
dns:
    provider: none
```

## 三、配置 kube-dns

### 1、上游 DNS服务器

_可用版本 v0.2.0_

默认情况下，kube-dns将使用已配置的DNS名称服务器(通常位于`/etc/resolv.conf`)来执行外部查询。如果希望配置特定的上游 DNS服务器供kube-dns使用，可以使用`upstreamnameservers`指令。

```yaml
dns:
    provider: kube-dns
    upstreamnameservers:
    - 1.1.1.1  
    - 8.8.4.4
```

## 2、CoreDNS (实验)

_可用版本 v0.2.0_

如果您想使用CoreDNS，您可以将`provider设置为CoreDNS`。CoreDNS还支持`node_selector`和`upstreamnameservers`参数设置。

```yaml
dns:
    provider: coredns
    upstreamnameservers:
    - 1.1.1.1
    - 8.8.4.4
```
