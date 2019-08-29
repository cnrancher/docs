---
title: 2 - 节点需求
weight: 2
aliases:
---

不管是单节点安装Rancher Server，或高可用安装Rancher Server,所有节点都需要满足以下的节点要求。

{{% tabs %}}
{{% tab 操作系统和Docker %}}

Rancher在以下操作系统及其后续的非主要发行版上受支持:

- Ubuntu 16.04.x (64-bit)
  - Docker 18.06.x, 18.09.x
- Ubuntu 18.04.x (64-bit)
  - Docker 18.06.x, 18.09.x
- RancherOS 1.3.x+ (64-bit)
  - Docker 18.06.x, 18.09.x
- Windows Server version 1803 (64-bit)
  - Docker 17.06

>1、Ubuntu、Centos操作系统有Desktop和Server版本，选择请安装server版本，`别自己坑自己！` \
>2、如果您正在使用RancherOS，请确保切换到受支持的Docker版本:\
`sudo ros engine switch docker-18.09.2`

{{% /tab %}}
{{% tab 硬件 %}}
硬件要求根据Rancher部署的K8S集群规模大小进行扩展，根据要求配置每个节点。

**HA安装需求(标准3节点)**

| **部署规模** | 集群数     | Nodes       | vCPUs                                              | RAM                                                |
| :----------- | :--------- | :---------- | :------------------------------------------------- | :------------------------------------------------- |
| 小           | 最多5个    | 最多50个    | 2                                                  | 8 GB                                               |
| 中           | 最多15个   | 最多200个   | 4                                                  | 16 GB                                              |
| 大           | 最多50个   | 最多500个   | 8                                                  | 32 GB                                              |
| 大+          | 最多100个  | 最多1000个  | 32                                                 | 128 GB                                             |
| 大++         | 超过100+个 | 超过1000+个 | [联系 Rancher](https://www.rancher.cn/contact/) | [联系 Rancher](https://www.rancher.cn/contact/) |



**单节点安装需求**

| **部署规模** | Clusters | Nodes     | vCPUs | RAM  |
| :----------- | :------- | :-------- | :---- | :--- |
| 小           | 最多5个  | 最多50个  | 4     | 8 GB |
| 中           | 最多15个 | 最多200个 | 8     | 16GB |

<br/>

{{% /tab %}}
{{% tab  "网络" %}}

<h2>节点IP地址</h2>

使用的每个节点（单节点安装，高可用性（HA）安装或集群中使用的worker节点）应配置静态IP。在DHCP的情况下，应配置DHCP IP保留以确保节点获得相同的IP分配。

<h2>端口需求</h2>

在HA集群中部署Rancher时，必须打开节点上的某些端口以允许与Rancher通信。必须打开的端口根据托管集群节点的计算机类型而变化，例如，如果要在基础结构托管的节点上部署Rancher，则必须为SSH打开`22`端口。下图描绘了需要为每种集群类型打开的端口。[集群类型]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning).

![Basic Port Requirements]({{< baseurl >}}/img/rancher/port-communications.svg)

{{< requirements_ports_rancher >}}
{{< requirements_ports_rke >}}
{{< ports_aws_securitygroup_nodedriver >}}
{{% /tab %}}
{{% /tabs %}}
