---
title: 2 - HA集群恢复
weight: 2
---

## 一、 恢复准备

1、需要在进行操作的主机上提前[安装RKE]({{< baseurl >}}/rke/latest/cn/installation/)、([RKE下载]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/download/rke/))和[安装kubectl]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/kubectl/)。 \
2、在开始还原之前，请确保已停止旧集群节点上的所有kubernetes服务。 \
3、建议创建三个全新节点作为集群恢复的目标节点。有关节点要求，请参阅[HA安装]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/)。您也可以使用现有节点，清除Kubernetes和Rancher配置，`这将破坏这些节点上的数据请做好备份`，点击了解[节点初始化]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/remove-node/)。

## 二、添加新ETCD节点并复制最新快照

假设集群中一个或者多个etcd节点发生故障，或者整个集群数据丢失，则需要进行etcd集群恢复。

**添加新ETCD节点并复制最新快照:**

1. 添加你添加的`全新ETCD节点`，可以是物理主机、本地虚拟机、云主机等；

1. 通过远程终端登录新主机；

1. 创建快照目录:

    ```bash
    mkdir -p /opt/rke/etcd-snapshots/
    ```

1. 复制备份的`最新快照`到/opt/rke/etcd-snapshots/目录

  - 如果是通过`rke0.2之前的版本`做的备份，需要拷贝`pki.bundle.tar.gz`到`/opt/rke/etcd-snapshots/`目录下；
  - 如果是使用`rke0.2以及之后的版本`做的备份，拷贝`xxx..rkestate`文件到rke 配置文件的当前目录下；

## 三、设置RKE配置文件

制作原始`rancher-cluster.yml`文件的副本

`cp rancher-cluster.yml rancher-cluster-restore.yml`

对副本配置文件进行以下修改:

- 删除或注释掉整个`addons:`部分，Rancher部署和设置配置已在etcd数据库中。
- 将`nodes:`部分更改为`全新ETCD节点`,注释掉其他节点。

`例: rancher-cluster-restore.yml`

```bash
nodes:
- address: 52.15.238.179     # 新etcd节点
  user: ubuntu
  role: [ etcd, controlplane, worker ]
# - address: 52.15.23.24
#   user: ubuntu
#   role: [ etcd, controlplane, worker ]
# - address: 52.15.238.133
#   user: ubuntu
#   role: [ etcd, controlplane, worker ]

# addons: |-
#   ---
#   kind: Namespace
#   apiVersion: v1
#   metadata:
#     name: cattle-system
#   ---
...
```

## 四、恢复ETCD数据

1、打开`shell终端`，切换到RKE二进制文件所在的目录，并且`上一步`修改的`rancher-cluster-restore.yml`文件也需要放在同一路径下。

2、根据系统类型，选择运行以下命令还原`etcd`数据：

```bash
# MacOS
./rke etcd snapshot-restore --name <snapshot>.db --config ./rancher-cluster-restore.yml
# Linux
./rke etcd snapshot-restore --name <snapshot>.db --config ./rancher-cluster-restore.yml
```

>RKE将在`新ETCD节点`上创建包含已还原数据的`ETCD`容器。此容器将保持运行状态，但无法完成etcd初始化并。

## 五、恢复集群

使用RKE并在`新ETCD节点`单节点上启动集群。根据系统类型，选择运行以下命令更新集群：

```bash
# MacOS
./rke up --config ./rancher-cluster-restore.yml
# Linux
./rke up --config ./rancher-cluster-restore.yml
```

## 六、测试集群

RKE运行完成后会创建`kubectl`的配置文件`kube_config_rancher-cluster-restore.yml`，可通过这个配置文件查询K8S集群节点状态：

```bash
kubectl --kubeconfig=kube_config_rancher-cluster-restore.yml  get nodes

NAME            STATUS    ROLES                      AGE       VERSION
52.15.238.179   Ready     controlplane,etcd,worker    1m       v1.10.5
18.217.82.189   NotReady  controlplane,etcd,worker   16d       v1.10.5
18.222.22.56    NotReady  controlplane,etcd,worker   16d       v1.10.5
18.191.222.99   NotReady  controlplane,etcd,worker   16d       v1.10.5
```

## 七、清理旧节点

通过kubectl从集群中删除旧节点

```bash
kubectl --kubeconfig=kube_config_rancher-cluster-restore.yml  delete node 18.217.82.189 18.222.22.56 18.191.222.99
```

## 八、重新启动全新ETCD节点

`全新ETCD节点`重启后，检查`Kubernetes Pods`的状态

```bash
kubectl --kubeconfig=kube_config_rancher-cluster-restore.yml  get pods --all-namespaces

NAMESPACE       NAME                                    READY     STATUS    RESTARTS   AGE
cattle-system   cattle-cluster-agent-766585f6b-kj88m    0/1       Error     6          4m
cattle-system   cattle-node-agent-wvhqm                 0/1       Error     8          8m
cattle-system   rancher-78947c8548-jzlsr                0/1       Running   1          4m
ingress-nginx   default-http-backend-797c5bc547-f5ztd   1/1       Running   1          4m
ingress-nginx   nginx-ingress-controller-ljvkf          1/1       Running   1          8m
kube-system     canal-4pf9v                             3/3       Running   3          8m
kube-system     cert-manager-6b47fc5fc-jnrl5            1/1       Running   1          4m
kube-system     kube-dns-7588d5b5f5-kgskt               3/3       Running   3          4m
kube-system     kube-dns-autoscaler-5db9bbb766-s698d    1/1       Running   1          4m
kube-system     metrics-server-97bc649d5-6w7zc          1/1       Running   1          4m
kube-system     tiller-deploy-56c4cf647b-j4whh          1/1       Running   1          4m
```

>直到Rancher服务器启动并且DNS/负载均衡器指向新集群，`cattle-cluster-agent和cattle-node-agent`pods将处于`Error或者CrashLoopBackOff`状态。

## 九、添加其他节点

1、编辑RKE配置文件`rancher-cluster-restore.yml`,添加或者取消其他节点的注释。

`例：rancher-cluster-restore.yml`

```bash
nodes:
- address: 52.15.238.179     # 新ETCD节点
  user: ubuntu
  role: [ etcd, controlplane, worker ]
- address: 52.15.23.24
  user: ubuntu
  role: [ etcd, controlplane, worker ]
- address: 52.15.238.133
  user: ubuntu
  role: [ etcd, controlplane, worker ]

# addons: |-
#   ---
#   kind: Namespace
...
```

2、更新集群

根据系统类型，选择运行以下命令更新集群：

```bash
# MacOS
./rke up --config ./rancher-cluster-restore.yml
# Linux
./rke up --config ./rancher-cluster-restore.yml
```