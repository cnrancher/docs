---
title: 1 - RKE集群备份
weight: 1
---

> **重要提示** 此方法直接使用RKE进行集群备份，它适用于RKE创建并导入的业务集群或者RKE部署的local集群

本节介绍在Rancher HA下如何备份数据。

- Rancher Kubernetes Engine v0.1.7或更高版本

    RKE v0.1.7以及更高版本才支持`etcd`快照功能

- `rancher-cluster.yml`

    需要使用到安装Rancher的RKE配置文件`rancher-cluster.yml`，将此文件需放在与RKE二进制文件同级目录中

## 一、创建ETCD数据快照

有两种方案创建`etcd`快照: 定时自动创建快照和或手动创建快照，每种方式对应特定的场景。

- 方案 A: 定时自动创建快照

    在Rancher HA安装后，我们建议配置RKE以定时(默认5分钟)自动创建快照，以便始终拥有可用的安全恢复点。

- 方案 B: 手动创建快照

    我们建议在升级或恢复其他快照等事件之前创建一次性快照。

### 方案 A: 定时自动创建快照

对于通过RKE高可用安装的Rancher，我们建议开启定时自动创建快照，以便始终拥有安全的恢复点。

定时自动创建快照服务是RKE附带的服务，默认没有开启。可以通过在`rancher-cluster.yml`中添加配置来启用etcd-snapshot(定时自动创建快照)服务。

**启用定时自动创建快照:**

1. 编辑`rancher-cluster.yml`配置文件；

2. 在`rancher-cluster.yml`配置文件中添加以下代码:

    _RKE v0.1.x_

    ```bash
    services:
      etcd:
        snapshot: true  # 是否启用快照功能，默认false；
        creation: 6h0s  # 快照创建间隔时间，不加此参数，默认5分钟；
        retention: 24h  # 快照有效期，此时间后快照将被删除；
    ```

    _RKE v0.2.0+_

    ```bash
    services:
      etcd:
        backup_config:
          enabled: true     # enables recurring etcd snapshots
          interval_hours: 6 # time increment between snapshots
          retention: 60     # time in days before snapshot purge
          # Optional S3
          s3_backup_config:
            access_key: "myaccesskey"
            secret_key:  "myaccesssecret"
            bucket_name: "my-backup-bucket"
            endpoint: "s3.eu-west-1.amazonaws.com"
            region: "eu-west-1"
    ```

3. 根据实际需求修改以上参数；

4. 保存并关闭`rancher-cluster.yml`；

5. 打开**Terminal**并切换路径到RKE二进制文件所在目录.确保`rancher-cluster.yml`也在这个路径下；

6. 运行以下命令:

    ```bash
    # MacOS
    ./rke_darwin-amd64 up --config rancher-cluster.yml
    # Linux
    ./rke_linux-amd64 up --config rancher-cluster.yml
    ```

>**结果:** RKE会在每个etcd节点上定时获取快照，并将快照将保存到每个etcd节点的:`/opt/rke/etcd-snapshots/`目录下

### 方案 B: 手动创建快照

>**警告** 1、在rke v0.2.0以前的版本，RKE将备份证书和配置文件到`pki.bundle.tar.gz`文件中，并保存在`/opt/rke/etcd-snapshots`目录中。通过v0.2.0之前的版本恢复系统时，需要快照和pki文件。\
2、从rke v0.2.0开始，因为架构调整不再需要`pki.bundle.tar.gz`文件，当rke 创建集群后，会在配置文件当前目录下生成`xxxx.rkestate`文件，文件中保存了集群的配置信息和各组件使用的证书信息。

**手动创建快照:**

1. 打开**Terminal**并切换路径到RKE二进制文件所在目录.确保`rancher-cluster.yml`也在该路径下

2. 输入以下命令:

    >注意:替换`<SNAPSHOT.db>`为您设置的快照名称，例如:<SNAPSHOT.db>

    ```bash
    # MacOS
    ./rke etcd snapshot-save --name <SNAPSHOT.db> --config rancher-cluster.yml
    # Linux
    ./rke etcd snapshot-save --name <SNAPSHOT.db> --config rancher-cluster.yml
    ```

>**结果:** RKE会获取每个`etcd`节点的快照，并保存在每个etcd节点的`/opt/rke/etcd-snapshots`目录下；

## 二、备份快照到安全位置

在创建快照后，应该把它保存到安全的地方，以便在集群遇到灾难情况时快照不受影响，这个位置应该是持久的。

复制`/opt/rke/etcd-snapshots`目录下所有文件到安全位置。

- 在rke v0.2.0以前的版本，备份`/opt/rke/etcd-snapshots`目录中的快照文件和`pki.bundle.tar.gz`文件，以及rke 配置文件到安全位置，通过v0.2.0之前的版本恢复系统时，需要这些文件。
- 在rke v0.2.0以及以后的版本，备份`/opt/rke/etcd-snapshots`目录中的快照文件和rke配置文件，以及配置文件当前目录下的`xxxx.rkestate`文件，通过v0.2.0之后版本恢复系统时，需要这些文件。
