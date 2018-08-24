---
title: 2 - 集群恢复
weight: 2
---

此节描述了如何在灾难情形下丢失Rancher数据时恢复etcd快照。

## 1. ETCD集群容错表

建议在ETCD集群中使用奇数个成员,通过添加额外成员可以获得更高的失败容错。在比较偶数和奇数大小的集群时，你可以在实践中看到这一点:

| 集群大小 | MAJORITY | 失败容错 |
| ------------ | -------- | ----------------- |
| 1            | 1        | 0                 |
| 2            | 2        | 0                 |
| 3            | 2        | **1**             |
| 4            | 3        | 1                 |
| 5            | 3        | **2**             |
| 6            | 4        | 2                 |
| 7            | 4        | **3**             |
| 8            | 5        | 3                 |
| 9            | 5        | **4**             |

## 2. 创建新节点并获取最新快照

假设集群中一个或者多个etcd节点发生故障，导致ectd集群挂掉， 则需要进行etcd集群恢复。然后将最新的etcd快照拷贝到该节点。

**创建新节点并获取最新快照:**

1. 创建你选择的新节点，可以是物理主机、本地虚拟机、云主机等；

2. 通过远程终端登录新主机；

3. 创建快照目录:

    ```bash
    mkdir -p /opt/rke/etcd-snapshots/
    ```

4. 复制备份的最新快照到每个`etcd`节点`/opt/rke/etcd-snapshots/`目录下:

    ```bash
    s3cmd get s3://rke-etcd-snapshots/<SNAPSHOT.db> /opt/rke/etcd-snapshots/<SNAPSHOT.db>
    ```

## 3. 恢复 `etcd` 数据

要还原`etcd`节点上的最新快照，请运行RKE命令`rke etcd snapshot-restore`。此命令将恢复`/opt/rke/etcd-snapshots`明确定义的快照。当你运行时`rke etcd snapshot-restore`，RKE会删除旧`etcd`容器(如果它仍然存在)。

>**警告:** 还原`etcd`快照会删除当前`etcd`集群并将其替换为新集群。在运行该`rke etcd snapshot-restore`命令之前，请备份当前集群中的所有重要数据。

### **先决条件**

- Rancher Kubernetes Engine 等于v0.1.7或更高版本

    RKE v0.1.7及更高版本才支持创建etcd快照功能。

- rancher-cluster.yml

    你需要把用于Rancher安装的RKE配置文件`rancher-cluster.yml`，放在与RKE二进制文件相同的目录中。

- 你必须将每个`etcd`节点还原到*同一*快照版本。在运行`etcd snapshot-restore`命令之前，将最新的快照从一个节点复制到其他节点。

1. 在笔记本或者其他远程电脑上用编辑器打开`rancher-cluster.yml`

2. 用新节点替换旧节点地址,假设旧节点为`3.3.3.3`,新节点为`4.4.4.4`:

    ```yaml
    nodes:
      - address: 1.1.1.1
        user: root
        role: [controlplane,etcd,worker]
        ssh_key_path: ~/.ssh/id_rsa
      - address: 2.2.2.2
        user: root
        role: [controlplane,etcd,worker]
        ssh_key_path: ~/.ssh/id_rsa
      - address: 4.4.4.4 #3.3.3.3旧节点
        user: root
        role: [controlplane,etcd,worker]
        ssh_key_path: ~/.ssh/id_rsa
    ```

3. 保存`rancher-cluster.yml`

4. 打开``shell终端``,切换路径到RKE二进制文件所在的位置，并且上一步修改的`rancher-cluster.yml`文件也需要放在同一路径下。

5. 根据系统类型，选择运行以下命令还原`etcd`数据库：

    ```bash
    # MacOS
    ./rke_darwin-amd64 etcd snapshot-restore --name <SNAPSHOT.db> --config rancher-cluster.yml
    # Linux
    ./rke_linux-amd64 etcd snapshot-restore --name <SNAPSHOT.db> --config rancher-cluster.yml
    ```

6. 根据系统类型，选择运行以下命令更新集群：

    ```bash
    # MacOS
    ./rke_darwin-amd64 up --config rancher-cluster.yml
    # Linux
    ./rke_linux-amd64 up --config rancher-cluster.yml
    ```

7. 最后，重新启动所有集群节点上的Kubernetes组件，以防止潜在的`etcd`冲突。在每个节点上运行以下命令：

    ```bash
    docker restart kube-apiserver kubelet kube-controller-manager kube-scheduler  kube-proxy
    docker ps | grep flannel | cut -f 1 -d " " | xargs docker restart
    docker ps | grep calico | cut -f 1 -d " " | xargs docker restart
    ```
