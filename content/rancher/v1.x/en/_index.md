---
title: Rancher v1.0 Docs
shortTitle: Rancher 1.0-EN
weight: 2
---

## Rancher概览
---

Rancher是一个开源的企业级容器管理平台。通过Rancher，企业再也不必自己使用一系列的开源软件去从头搭建容器服务平台。Rancher提供了在生产环境中使用的管理Docker和Kubernetes的全栈化容器部署与管理平台。

Rancher由以下四个部分组成:

### 基础设施编排

Rancher可以使用任何公有云或者私有云的Linux[主机]({{< baseurl >}}/rancher/v1.x/cn/infrastructure/hosts/)资源。Linux主机可以是虚拟机，也可以是物理机。Rancher仅需要主机有CPU，内存，本地磁盘和网络资源。从Rancher的角度来说，一台云厂商提供的云主机和一台自己的物理机是一样的。

Rancher为运行容器化的应用实现了一层灵活的[基础设施服务]({{< baseurl >}}/rancher/v1.x/cn/rancher-services/)。Rancher的基础设施服务包括[网络]({{< baseurl >}}/rancher/v1.x/cn/rancher-services/networking)， [存储]({{< baseurl >}}/rancher/v1.x/cn/rancher-services/storage-service/)， [负载均衡]({{< baseurl >}}/rancher/v1.x/cn/rancher-services/load-balancer/)， [DNS]({{< baseurl >}}/rancher/v1.x/cn/rancher-services/dns-service/)和安全模块。Rancher的基础设施服务也是通过容器部署的，所以同样Rancher的基础设施服务可以运行在任何Linux主机上。

### 容器编排与调度

很多用户都会选择使用容器编排调度框架来运行容器化应用。Rancher包含了当前全部主流的编排调度引擎，例如[Docker Swarm]({{< baseurl >}}/rancher/v1.x/cn/infrastructure/swarm)， [Kubernetes]({{< baseurl >}}/rancher/v1.x/cn/kubernetes)， 和[Mesos]({{< baseurl >}}/rancher/v1.x/cn/infrastructure/mesos/)。同一个用户可以创建Swarm或者Kubernetes集群。并且可以使用原生的Swarm或者Kubernetes工具管理应用。

除了Swarm，Kubernetes和Mesos之外，Rancher还支持自己的Cattle容器编排调度引擎。Cattle被广泛用于编排Rancher自己的基础设施服务以及用于Swarm集群，Kubernetes集群和Mesos集群的配置，管理与升级。

### 应用商店

Rancher的用户可以在[应用商店]({{< baseurl >}}/rancher/v1.x/cn/catalog)里一键部署由多个容器组成的应用。用户可以管理这个部署的应用，并且可以在这个应用有新的可用版本时进行自动化的升级。Rancher提供了一个由Rancher社区维护的应用商店，其中包括了一系列的流行应用。Rancher的用户也可以[创建自己的私有应用商店]({{< baseurl >}}/rancher/v1.x/cn/configuration/catalog/private-catalog/)。

### 企业级权限管理

Rancher支持灵活的插件式的用户认证。支持Active Directory，LDAP， Github等 [认证方式]({{< baseurl >}}/rancher/v1.x/cn/configuration/access-control/)。 Rancher支持在[环境]({{< baseurl >}}/rancher/v1.x/cn/configuration/environments/)级别的基于角色的访问控制 (RBAC)，可以通过角色来配置某个用户或者用户组对开发环境或者生产环境的访问权限。

下图展示了Rancher的主要组件和功能:

<img src="/img/rancher/rancher.png" width="800" alt="Rancher Overview">
