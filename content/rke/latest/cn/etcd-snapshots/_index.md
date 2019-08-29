---
title: 5 - 备份和恢复
weight: 5
aliases:
---

> v0.1.7版本可用

RKE集群可以设置成自动创建etcd快照，在灾难发生时可以通过这些快照进行恢复，这些快照保存在本地`/opt/rke/etcd-snapshot`目录。

> v0.2.0版本可用

RKE还可以将快照上传到S3兼容的后端。此外`pki.bundle.tar.gz`文件不再需要，因为v0.2.0使用`xxxx.rkestate`来[存储Kubernetes集群状态]({{< baseurl >}}/rke/latest/cn/installation/#八-kubernetes集群状态文件)。

## 一、动手创建快照

`rke etcd snapshot-save`命令将从集群中的每个etcd节点保存etcd快照，快照保存在`/opt/rke/etcd-snapshots`中。运行该命令时，将创建一个附加容器来获取快照。快照完成后，容器将自动删除。

在v0.2.0之前，RKE保存了证书的备份，名为`pki.bundle.tar.gz`，也保存在`/opt/rke/etcd-snapshots`中。在v0.2.0之前的版本中，系统恢复需要快照和`pki包`文件。

### 1、rke etcd snapshot-save 命令选项

| 选项 | 描述 | S3特有 |
| --- | --- | --- |
|   `--name` value         |    指定快照名称 |  |
|   `--config` value       |    指定RKE集群YAML文件(default: "cluster.yml") [$RKE_CONFIG] |  |
|   `--s3`                 |    启用s3备份|   * |
|   `--s3-endpoint` value  |    指定s3备份地址 (default: "s3.amazonaws.com") |   * |
|   `--access-key` value   |    指定s3 accessKey |   * |
|   `--secret-key` value   |    指定s3 secretKey |  * |
|   `--bucket-name` value  |    指定s3 bucket name |   * |
|   `--region` value       |    指定s3 bucket位置 (可选) |   * |
|   `--ssh-agent-auth`      |   [使用SSH Agent认证]({{< baseurl >}}/rke/latest/cn/config-options/#ssh-agent) | |
|   `--ignore-docker-version`  | [禁止Docker版本检查]({{< baseurl >}}/rke/latest/cn/config-options/#supported-docker-versions) | |

### 2、本地手动快照示例

```bash
rke etcd snapshot-save --config <cluster.yml> --name <snapshot-name>
```

快照将保存在:  `/opt/rke/etcd-snapshots`

### 3、手动快照上传到S3示例

> v0.2.0版本可用

```bash
rke etcd snapshot-save --config cluster.yml --name <snapshot-name>  \
--s3 --access-key <S3_ACCESS_KEY> --secret-key <S3_SECRET_KEY> \
--bucket-name <s3-bucket-name> --s3-endpoint <s3.amazonaws.com>
```

快照将保存在:`/opt/rke/etcd-snapshots`，同时也将上传到S3。

## 二、自动定时快照

需要使用额外的配置选项启用etcd-snapshot服务，默认情况下，etcd-snapshot服务为具有`etcd角色`的每个节点获取快照，并将它们存储到本地磁盘`/opt/rke/etcd-snapshot`目录中。如果有配置S3相关参数，快照也将被上传到S3存储后端。

当集群启用了`etcd-snapshot`服务时，可以查看`etcd-roll-snapshot`容器日志，以确认是否自动创建备份。

```bash
docker logs etcd-rolling-snapshots

time="2018-05-04T18:39:16Z" level=info msg="Initializing Rolling Backups" creation=1m0s retention=24h0m0s
time="2018-05-04T18:40:16Z" level=info msg="Created backup" name="2018-05-04T18:40:16Z_etcd" runtime=108.332814ms
time="2018-05-04T18:41:16Z" level=info msg="Created backup" name="2018-05-04T18:41:16Z_etcd" runtime=92.880112ms
time="2018-05-04T18:42:16Z" level=info msg="Created backup" name="2018-05-04T18:42:16Z_etcd" runtime=83.67642ms
time="2018-05-04T18:43:16Z" level=info msg="Created backup" name="2018-05-04T18:43:16Z_etcd" runtime=86.298499ms
```

### 1、Etcd快照服务选项

根据您的RKE版本，用于配置自动定时快照的选项可能有所不同。

- v0.2.0以及之后版本

    |选项|描述| S3特有 |
    |---|---| --- |
    |**interval_hours**| 重复备份之间的持续时间（以小时为单位）。该参数取代了`creation`参数，如果同时设置`creation`选项，`creation`设置将被覆盖。| |
    |**retention**| 备份轮换前要保留的快照数。该参数取代了`retention`参数，并且如果如果同时设置两个参数，`retention`参数将被覆盖。| |
    |**bucket_name**| S3存储`bucket`名称| * |
    |**access_key**| S3 access key | * |
    |**secret_key** |S3 secret key | * |
    |**region** |S3 region  可选| * |
    |**endpoint** |S3 regions endpoint | * |

    ```yaml
    services:
      etcd:
        backup_config:
          interval_hours: 12
          retention: 6
          s3backupconfig:
            access_key: S3_ACCESS_KEY
            secret_key: S3_SECRET_KEY
            bucket_name: s3-bucket-name
            region: ""
            endpoint: s3.amazonaws.com
    ```

- v0.2.0之前

    |选项|描述|
    |---|---|
    |**Snapshot**|默认情况下，禁用定时快照服务。要启用该服务，需要在`etcd服务`中将其设置为`true`。|
    |**Creation**|默认情况下，快照服务将每隔5分钟创建一次（5m0s）。|
    |**Retention**|默认情况下，所有快照保留24小时。|

    ```yaml
    services:
      etcd:
        snapshot: true
        creation: 5m0s
        retention: 24h
    ```

## 三、Etcd灾难恢复

>**重要提示:** rke 0.1.x与rke 0.2.x因为[Kubernetes集群状态的存储方式]({{< baseurl >}}/rke/latest/cn/installation/#八-kubernetes集群状态文件)的改变，如果集群是通过rke 0.1.x做的备份，那么无法直接使用rke 0.2.x进行数据恢复。如果集群状态正常，可以在恢复之前运行`rke up --config cluster.yml`进行集群状态更新。如果集群状态不正常，那只能通过rke 0.1.x进行恢复。
>
>**警告:** 恢复etcd快照会删除当前的etcd集群并将其替换为新集群。在运行该`rke etcd snapshot-restore`命令之前，应备份集群中的所有重要数据。

### 1、`rke etcd snapshot-restore`选项

| Option | Description | S3 Specific |
| --- | --- | ---|
| `--name` value            |  指定快照名称 | |
| `--config` value          |  指定RKE集群YAML文件(default: "cluster.yml") [$RKE_CONFIG] | |
| `--s3`                    |  启用s3备份 |* |
| `--s3-endpoint` value     |  指定s3备份地址(default: "s3.amazonaws.com") | * |
| `--access-key` value      |  指定 s3 accessKey | *|
| `--secret-key` value      |  指定 s3 secretKey | *|
| `--bucket-name` value     |  指定 s3 bucket name | *|
| `--region` value          |  指定s3 bucket位置 (可选) | *|
| `--ssh-agent-auth`      |   [使用SSH Agent认证]({{< baseurl >}}/rke/latest/cn/config-options/#ssh-agent) | |
| `--ignore-docker-version`  | [禁止Docker版本检查]({{< baseurl >}}/rke/latest/cn/config-options/#supported-docker-versions) |

### 2、从本地快照还原示例

当从本地快照恢复etcd时，假设快照位于`/opt/rke/etcd-snapshots`中。在rke v0.2.0之前的版本中，`pki.bundle.tar.gz`文件也应该在相同的位置, 从v0.2.0开始，不再需要这个文件，因为v0.2.0改变了[Kubernetes集群状态的存储方式]({{< baseurl >}}/rke/latest/cn/installation/#八-kubernetes集群状态文件)。

```bash
rke etcd snapshot-restore --config cluster.yml --name mysnapshot
```

### 3、从S3中还原快照示例

> v0.2.0版本可用

当从S3中的快照恢复etcd时，命令需要S3信息，以便连接到S3后端并检索快照。

```bash
rke etcd snapshot-restore --config cluster.yml --name snapshot-name \
--s3 --access-key S3_ACCESS_KEY --secret-key S3_SECRET_KEY \
--bucket-name s3-bucket-name --s3-endpoint s3.amazonaws.com
```

## 四、备份与恢复示例

在本例中，Kubernetes集群部署在两个AWS节点上。

|  Name |    IP    |          Role          |
|:-----:|:--------:|:----------------------:|
| node1 | 10.0.0.1 | [controlplane, worker] |
| node2 | 10.0.0.2 | [etcd]                 |

### 1、备份`etcd`集群

以Kubernetes集群的本地快照为例。从v0.2.0开始，您还可以使用[S3选项](#options-for-rke-etcd-snapshot-save)将此快照直接上传到S3后端。

```bash
rke etcd snapshot-save --name snapshot.db --config cluster.yml
```

![etcd snapshot]({{< baseurl >}}/img/rke/rke-etcd-backup.png)

### 2、将快照存储在S3

从v0.2.0开始，不再需要这个步骤，因为RKE在运行`rke etcd snapshot-save`命令时，可以通过添加[S3选项](#options-for- RKE -etcd-snapshot-save)自动从S3上传和下载快照。

在`node2`上获取etcd快照之后，我们建议将该备份保存在持久性位置。其中一个选项是保存快照文件和`pki.bundle.tar.gz`文件到S3。

> **注意:** 从v0.2.0开始，恢复过程不再需要`pki.bundle.tar.gz`文件。

```bash
# If you're using an AWS host and have the ability to connect to S3
root@node2:~# s3cmd mb s3://rke-etcd-backup
root@node2:~# s3cmd /opt/rke/etcd-snapshots/snapshot.db /opt/rke/etcd-snapshots/pki.bundle.tar.gz s3://rke-etcd-backup/
```

### 3、将备份放在新节点上

为了模拟失败场景，我们关闭`node2`。

```bash
root@node2:~# poweroff
```

|  Name |    IP    |          Role          |
|:-----:|:--------:|:----------------------:|
| node1 | 10.0.0.1 | [controlplane, worker] |
| ~~node2~~ | ~~10.0.0.2~~ | ~~[etcd]~~                 |
| node3 | 10.0.0.3 | [etcd]                 |
|   |   |   |

在恢复etcd并运行`rke up`之前，我们需要检索保存在S3上的备份到一个新节点，例如: `node3`。从v0.2.0开始，您可以在运行`restore`命令时直接从S3检索快照，所以这一步是针对那些没有使用集成S3选项而在外部存储快照的用户。

```bash
# Make a Directory
root@node3:~# mkdir -p /opt/rke/etcdbackup
# Get the Backup from S3
root@node3:~# s3cmd get s3://rke-etcd-backup/snapshot.db /opt/rke/etcd-snapshots/snapshot.db
# Get the pki bundle from S3, only needed prior to v0.2.0
root@node3:~# s3cmd get s3://rke-etcd-backup/pki.bundle.tar.gz /opt/rke/etcd-snapshots/pki.bundle.tar.gz
```

### 4、通过备份恢复etcd到新节点上

在更新和恢复etcd之前，您需要编辑`cluster.yml`配置文件，添加新节点并设置为`etcd`角色，注释掉所有旧节点。

```yaml
nodes:
    - address: 10.0.0.1
      hostname_override: node1
      user: ubuntu
      role:
        - controlplane
        - worker
#    - address: 10.0.0.2
#      hostname_override: node2
#      user: ubuntu
#      role:
#       - etcd
    - address: 10.0.0.3
      hostname_override: node3
      user: ubuntu
      role:
        - etcd
```

将新节点添加到`cluster.yml`之后。运行`rke etcd snapshot-restore`从备份启动`etcd`。快照和`pki.bundle.tar.gz`文件需要预先放在`/opt/rke/etcd-snapshots`目录中。

从v0.2.0开始，如果您想直接从S3检索快照，请添加[S3选项](#options-for-rke-etcd-snapshot-restore)。

> **注意:**从v0.2.0开始，恢复过程不再需要`pki.bundle.tar.gz`文件。

```bash
rke etcd snapshot-restore --name snapshot.db --config cluster.yml
```

最后，我们需要恢复集群上的操作是让Kubernetes API指向新的`etcd`，方法是使用新的`cluster.yml`再次运行`rke up`

```bash
rke up --config cluster.yml
```

通过检查集群上的pod，来确认Kubernetes集群是否正常工作。

```bash
> kubectl --kubeconfig=kube_configxxx.yml  get  pods
NAME                     READY     STATUS    RESTARTS   AGE
nginx-65899c769f-kcdpr   1/1       Running   0          17s
nginx-65899c769f-pc45c   1/1       Running   0          17s
nginx-65899c769f-qkhml   1/1       Running   0          17s
```

## 五、故障排除

从`v0.1.9`开始，在恢复成功和失败时都会删除`rke-bundle-cert`容器。要调试任何问题，您需要查看从rke生成的`日志文件`。

在`v0.1.8`以及早期版本中，`rke-bundle-cert`容器是etcd恢复失败后遗留下来的。如果您在还原`etcd快照`时遇到问题，那么在尝试进行另一次还原之前，您可以在每个etcd节点上执行以下操作:

```bash
docker container rm --force rke-bundle-cert
```

当`etcd`备份或恢复成功时，通常会删除`rke-bundle-cert`容器。无论什么时候出现问题，都会留下`rke-bundle-cert`容器。您可以查看日志或检查容器，看看问题出在哪里。

```bash
docker container logs --follow rke-bundle-cert
docker container inspect rke-bundle-cert
```

需要注意的重要一点是容器的挂载和`pki.bundl .tar.gz`的位置。