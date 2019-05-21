---
title: 2 - RKE安装Kubernetes
weight: 2
---

使用RKE以高可用etcd配置安装Kubernetes。

> **注意:** 离线环境，请访问[离线安装]({{< baseurl >}}/rancher/v2.x/cn/installation/air-gap-installation/) 。

## 一、创建`rancher-cluster.yml`文件

使用下面的示例创建`rancher-cluster.yml`文件，使用创建的3个节点的IP地址或域名替换列表中的IP地址。

> **注意:**  如果节点有``公网地址 和 内网地址``地址，建议手动设置`internal_address:`以便Kubernetes将内网地址用于集群内部通信。如果需要开启自动配置安全组或防火墙，某些服务(如AWS EC2)需要设置`internal_address:`。

```yaml
nodes:
  - address: 165.227.114.63
    internal_address: 172.16.22.12
    user: ubuntu
    role: [controlplane,worker,etcd]
  - address: 165.227.116.167
    internal_address: 172.16.32.37
    user: ubuntu
    role: [controlplane,worker,etcd]
  - address: 165.227.127.226
    internal_address: 172.16.42.73
    user: ubuntu
    role: [controlplane,worker,etcd]

services:
  etcd:
    snapshot: true
    creation: 6h
    retention: 24h
```

### 1、常规RKE节点选项

| Option | Required | Description |
| --- | --- | --- |
| `address` | yes | 公共域名或IP地址 |
| `user` | yes | 可以运行docker命令的用户|
| `role` | yes | 分配给节点的Kubernetes角色列表 |
| `internal_address` | no | 内部集群通信的私有域名或IP地址 |
| `ssh_key_path` | no | 用于对节点进行身份验证的SSH私钥的路径（默认为~/.ssh/id_rsa） |

### 2、高级配置

RKE有许多配置选项可用于自定义安装以适合您的特定环境。

有关选项和功能的完整列表，请查看[RKE文档]({{< baseurl >}}/rke/latest/cn/config-options/) 。

## 二、创建Kubernetes集群

运行RKE命令创建Kubernetes集群

```bash
rke up --config ./rancher-cluster.yml
```

完成后，它应显示：`Finished building Kubernetes cluster successfully`。

## 三、测试集群

RKE应该已经创建了一个文件`kube_config_rancher-cluster.yml`。这个文件包含kubectl和helm访问K8S的凭据。

>**注意:** 如果你使用的文件不叫`rancher-cluster.yml`, 那么这个`kube config`配置文件将被命名为`kube_config_<FILE_NAME>.yml`。

您可以将此文件复制到`$HOME/.kube/config`，或者如果你正在使用多个Kubernetes集群，请将`KUBECONFIG`环境变量设置为`kube_config_rancher-cluster.yml`文件路径。

```bash
export KUBECONFIG=$(pwd)/kube_config_rancher-cluster.yml
```

通过`kubectl`测试您的连接，并查看您的所有节点是否处于`Ready`状态。

```bash
kubectl --kubeconfig=kube_configxxx.yml  get  nodes

NAME                          STATUS    ROLES                      AGE       VERSION
165.227.114.63                Ready     controlplane,etcd,worker   11m       v1.10.1
165.227.116.167               Ready     controlplane,etcd,worker   11m       v1.10.1
165.227.127.226               Ready     controlplane,etcd,worker   11m       v1.10.1
```

## 四、检查集群Pod的运行状况

- Pods是`Running`或者`Completed`状态。
- `READY`列显示所有正在运行的容器 (i.e. `3/3`)，`STATUS`显示POD是`Running`。
- Pods的`STATUS`是`Completed`为`run-one Jobs`,这些pods`READY`应该为`0/1`。

```bash
kubectl --kubeconfig=kube_configxxx.yml  get  pods --all-namespaces

NAMESPACE       NAME                                      READY     STATUS      RESTARTS   AGE
ingress-nginx   nginx-ingress-controller-tnsn4            1/1       Running     0          30s
ingress-nginx   nginx-ingress-controller-tw2ht            1/1       Running     0          30s
ingress-nginx   nginx-ingress-controller-v874b            1/1       Running     0          30s
kube-system     canal-jp4hz                               3/3       Running     0          30s
kube-system     canal-z2hg8                               3/3       Running     0          30s
kube-system     canal-z6kpw                               3/3       Running     0          30s
kube-system     kube-dns-7588d5b5f5-sf4vh                 3/3       Running     0          30s
kube-system     kube-dns-autoscaler-5db9bbb766-jz2k6      1/1       Running     0          30s
kube-system     metrics-server-97bc649d5-4rl2q            1/1       Running     0          30s
kube-system     rke-ingress-controller-deploy-job-bhzgm   0/1       Completed   0          30s
kube-system     rke-kubedns-addon-deploy-job-gl7t4        0/1       Completed   0          30s
kube-system     rke-metrics-addon-deploy-job-7ljkc        0/1       Completed   0          30s
kube-system     rke-network-plugin-deploy-job-6pbgj       0/1       Completed   0          30s
```

## 五、保存配置文件

保存`kube_config_rancher-cluster.yml`和`rancher-cluster.yml`文件的副本,您将需要这些文件来维护和升级Rancher实例。

## 六、Issues or errors

查看[Troubleshooting](./troubleshooting/)页面。

## [下一步: 初始化Helm(Install tiller)](../helm-install/)
