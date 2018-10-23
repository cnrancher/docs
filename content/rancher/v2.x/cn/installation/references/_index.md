---
title: 4 - 端口需求
weight: 4
---

要保证Rancher正常运行，需要主机或者安全策略打开以下端口。使用云服务创建集群(如Amazon EC2或DigitalOcean),Rancher会自动打开这些端口。下图显示了Rancher的基本端口要求。如果需要了解更多，请查阅下表。

![Basic port Requirements]({{< baseurl >}}/img/rancher/port-communications.svg)

## Rancher节点

The following table lists the ports that need to be open to and from nodes that are running the Rancher server container for [single node installs]({{< baseurl >}}/rancher/v2.x/en/installation/single-node-install/) or pods for [high availability installs]({{< baseurl >}}/rancher/v2.x/en/installation/ha-server-install/).

{{< ports-rancher-nodes >}}

## Kubernetes集群节点

The ports required to be open for cluster nodes changes depending on how the cluster was launched. Each of the tabs below list the ports that need to be opened for different [cluster creation options]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/#cluster-creation-options).

>**Tip:**
>
>If security isn't a large concern and you're okay with opening a few additional ports, you can use the table in [Commonly Used Ports](#commonly-used-ports) as your port reference instead of the comprehensive tables below.

{{% tabs %}}

{{% tab "Node Pools" %}}

The following table depicts the port requirements for [Rancher Launched Kubernetes]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/rke-clusters/) with nodes created in an [Infrastructure Provider]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/rke-clusters/node-pools/).

>**Note:**
>The required ports are automatically opened by Rancher during creation of clusters in cloud providers like Amazon EC2 or DigitalOcean.

{{< ports-iaas-nodes >}}

{{% /tab %}}

{{% tab "Custom Nodes" %}}

The following table depicts the port requirements for [Rancher Launched Kubernetes]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/rke-clusters/) with [Custom Nodes]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/rke-clusters/custom-nodes/).

{{< ports-custom-nodes >}}

{{% /tab %}}

{{% tab "Hosted Clusters" %}}

The following table depicts the port requirements for [hosted clusters]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/hosted-kubernetes-clusters). 

{{< ports-imported-hosted >}}

{{% /tab %}}

{{% tab "Imported Clusters" %}}

The following table depicts the port requirements for [imported clusters]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/imported-clusters/).

{{< ports-imported-hosted >}}

{{% /tab %}}

{{% /tabs %}}

## 其他端口注意事项

### 常用端口

这些端口通常需要在Kubernetes节点上打开，而不管它是什么类型的集群。

| Protocol |       Port       | Description                                     |
|:--------:|:----------------:|-------------------------------------------------|
|    TCP   |        22        | Node driver SSH provisioning                    |
|    TCP   |       2376       | Node driver Docker daemon TLS port              |
|    TCP   |       2379       | etcd client requests                            |
|    TCP   |       2380       | etcd peer communication                         |
|    UDP   |       8472       | Canal/Flannel VXLAN overlay networking          |
|    TCP   |       9099       | Canal/Flannel livenessProbe/readinessProbe      |
|    TCP   |       10250      | kubelet API                                     |
|    TCP   |       10254      | Ingress controller livenessProbe/readinessProbe |
| TCP/UDP  | 30000-</br>32767 | NodePort port range                             |

### 本地节点流量

标记为`local traffic`的端口(即在上述要求中，Kubernetes healthchecking (`livenessProbe` and`readinessProbe`)使用。
这些healthcheck是在节点本身上执行的。在大多数云环境中，默认情况下允许本地通信。

然而，当以下情况出现时，该流量可能会被阻塞:

- 节点上应用了严格的主机防火墙策略。
- 节点具有多个接口(multihomed)。

在这些情况下，您必须在您的主机防火墙中允许这类流量，或者在您的安全组配置中，在公共/私有云托管主机(如AWS或OpenStack)中允许这类流量。

### Rancher AWS EC2安全组

使用[AWS EC2 node driver]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/rke-clusters/node-pools/ec2/)提供Rancher中的集群节点，您可以选择让Rancher创建一个名为`rancher-nodes`的安全组，以下规则将自动添加到此安全组。

|       Type      | Protocol |  Port Range | Source/Destination     | Rule Type |
|-----------------|:--------:|:-----------:|------------------------|:---------:|
|       SSH       |    TCP   | 22          | 0.0.0.0/0              | Inbound   |
|       HTTP      |    TCP   | 80          | 0.0.0.0/0              | Inbound   |
| Custom TCP Rule |    TCP   | 443         | 0.0.0.0/0              | Inbound   |
| Custom TCP Rule |    TCP   | 2376        | 0.0.0.0/0              | Inbound   |
| Custom TCP Rule |    TCP   | 2379-2380   | sg-xxx (rancher-nodes) | Inbound   |
| Custom UDP Rule |    UDP   | 4789        | sg-xxx (rancher-nodes) | Inbound   |
| Custom TCP Rule |    TCP   | 6443        | 0.0.0.0/0              | Inbound   |
| Custom UDP Rule |    UDP   | 8472        | sg-xxx (rancher-nodes) | Inbound   |
| Custom TCP Rule |    TCP   | 10250-10252 | sg-xxx (rancher-nodes) | Inbound   |
| Custom TCP Rule |    TCP   | 10256       | sg-xxx (rancher-nodes) | Inbound   |
| Custom TCP Rule |    TCP   | 30000-32767 | 30000-32767            | Inbound   |
| Custom UDP Rule |    UDP   | 30000-32767 | 30000-32767            | Inbound   |
| All traffic     |    All   | All         | 0.0.0.0/0              | Outbound  |
