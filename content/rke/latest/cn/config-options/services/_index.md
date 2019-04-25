---
title: 6 - Kubernetes Services
weight: 6
---

部署Kubernetes, RKE在节点上以Docker容器方式部署几个核心组件或服务，根据节点的角色，部署的容器不同。

**所有服务都支持额外的[自定义参数、Docker卷和额外的环境变量]({{< baseurl >}}/rke/latest/cn/config-options/services/services-extras/).**

## 一、etcd

- Etcd是一个可靠、一致且分布式的键值存储，Kubernetes使用[etcd](https://github.com/coreos/etcd/blob/master/Documentation/docs.md)作为集群状态和数据的存储；
- RKE支持在`单节点模式或HA集群模式`下运行etcd,它还支持向集群`添加和删除`etcd节点；
- 可以启用etcd[自动定时快照]({{< baseurl >}}/rke/latest/cn/etcd-snapshots/#recurring-snapshots)功能，这些快照可用于恢复etcd集群；
- 默认情况下，RKE将全新部署etcd服务。但也可以使用已有的[外部etcd]({{< baseurl >}}/rke/latest/cn/config-options/services/external-etcd/)服务运行Kubernetes 。

## 二、Kubernetes API Server

> **Rancher 2用户注意事项** 如果在创建[Rancher Launched Kubernetes]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/rke-clusters/)时使用[Config File]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/rke-clusters/options/#config-file) 配置集群选项，服务的名称应该只包含下划线，比如: `kube_api`，这仅适用于Rancher v2.0.5和v2.0.6。

[Kubernetes API](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)REST服务，它处理所有Kubernetes对象的请求和数据，并为其他Kubernetes组件提供共享状态。

```yaml
services:
  kube-api:
    # 在Kubernetes上创建服务的IP范围
    # 这个值必须与kube-controller中的service_cluster_ip_range相同
    service_cluster_ip_range: 10.43.0.0/16
    # NodePort服务映射端口范围
    service_node_port_range: 30000-32767
    pod_security_policy: false
    # 启用总是拉取镜像
    # 可用版本 v0.2.0
    always_pull_images: false
```

### 1、Kubernetes API Server设置选项

RKE支持`kube-api`服务的以下设置选项:

- **Service Cluster IP Range** (`service_cluster_ip_range`)

    这是将分配给Kubernetes创建的服务的虚拟IP地址。默认情况下，`cluster IP`范围是`10.43.0.0/16`。如果要更改此值，必须在Kubernetes控制器管理器(kube-controller)服务上配置相同的值。

- **Node Port Range** (`service_node_port_range`)

    使用NodePort[类型](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)创建的Kubernetes服务的端口范围，默认情况下，端口范围为`30000-32767`。

- **Pod Security Policy** (`pod_security_policy`)

    启用[Kubernetes Pod安全策略](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)选项，默认不启用pod安全策略(`pod_security_policy: false`)。

    > **注意:** 如果将`pod_security_policy`值设置为`true`, RKE将配置一个开放策略，允许所有Pod在集群上工作，您需要配置自己的策略来充分利用PSP(pod_security_policy)。

- **Always Pull Images** (`always_pull_images`)

    启用总是spullimages承认控制器插件。启用`AlwaysPullImages`是一种安全最佳实践，它强制Kubernetes验证镜像并使用远程镜像仓库获取凭据。仍然使用本地镜像层缓存，但是在启动容器时拉取和比较镜像散列时，确实增加了一点开销。

    >注意:从v0.2.0开始可用

## 三、Kubernetes Controller Manager

> **Rancher 2用户注意事项** 如果在创建[Rancher Launched Kubernetes]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/rke-clusters/)时使用[Config File]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/rke-clusters/options/#config-file) 配置集群选项，服务的名称应该只包含下划线，比如: `kube_controller`，这仅适用于Rancher v2.0.5和v2.0.6。

[Kubernetes Controller Manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)服务是负责运行Kubernetes主控制循环的组件。控制器管理器通过Kubernetes API服务器监视集群所需的状态，并对当前状态进行必要的更改以达到所需的状态。

```yaml
services:
    kube-controller:
      # 集群中的pod分配IP地址池
      cluster_cidr: 10.42.0.0/16
      # 在Kubernetes上创建的服务的IP范围
      # 这个值必须与kube-api中的service_cluster_ip_range相同
      service_cluster_ip_range: 10.43.0.0/16
```

### 1、Kubernetes Controller Manager Options

RKE支持`kube-controller`服务的以下设置选项:

- **Cluster CIDR** (`cluster_cidr`)

    CIDR池用于为集群中的pod分配IP地址，默认情况下，`集群中的每个节点都从这个池中分配一个(/24)位的子网来分配ip给pod`，这个选项的默认值是`10.42.0.0/16`。

- **Service Cluster IP Range** (`service_cluster_ip_range`)
  
    这将分配给Kubernetes上创建的服务的虚拟IP地址。默认情况下， cluster IP范围是`10.43.0.0/16`。如果要更改此值，必须在Kubernetes控制器管理器(`kube-api`)服务上配置相同的值。

## 四、Kubelet

[kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)服务充当Kubernetes的`节点代理`，它运行在RKE部署的所有节点上，并使Kubernetes能够管理节点上的容器运行时。

```yaml
services:
    kubelet:
     # 集群的搜索域
     cluster_domain: cluster.local
     # DNS服务IP地址
     cluster_dns_server: 10.43.0.10
     # 禁止swap
     fail_swap_on: false
```

### 1、Kubelet 设置选项

RKE支持`kubelet`服务的以下设置选项:

- **Cluster Domain** (`cluster_domain`)
  
    集群的[base domain](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)，默认情况下，被设置为`cluster.local`。

- **Cluster DNS Server** (`cluster_dns_server`)
  
    分配给集群内DNS服务IP地址。DNS查询将发送到KubeDNS使用的这个IP地址，这个选项的默认值是`10.43.0.10`

- **Fail if Swap is On** (`fail_swap_on`)
  
    在Kubernetes中，如果节点上启用了`swap`, kubelet的启动将报错。RKE`不遵循`此缺省值，允许在启用交换的节点上部署。默认情况下，值为`false`。如果您想恢复到默认的kubelet设置，请将此选项设置为`true`。

## 五、Kubernetes Scheduler

[Kubernetes Scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)服务负责根据各种配置、指标、资源需求和特定于工作负载的需求调度集群工作负载。目前，RKE不支持`scheduler`服务的任何特定设置选项。

## 六、Kubernetes Network Proxy

[Kubernetes network proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)服务运行在所有节点上，并管理Kubernetes为TCP/UDP端口创建的端点。目前，RKE不支持kube proxy服务的任何特定设置选项。