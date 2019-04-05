---
title: 5 - 备份和恢复
weight: 5
aliases:
---

> v0.1.7版本可用

RKE集群可以设置成自动创建etcd快照，在灾难发生时可以通过这些快照进行恢复，这些快照保存在本地`/opt/rke/etcd-snapshot`目录。

> v0.2.0版本可用

RKE还可以将快照上传到S3兼容的后端。此外,**pki.bundle.tar.gz**文件不再需要，因为v0.2.0使用[存储Kubernetes集群状态]({{< baseurl >}}/rke/v0.1.x/en/installation/#kubernetes-cluster-state)保存证书。

## 一次性快照

`rke etcd snapshot-save`命令将从集群中的每个etcd节点保存etcd快照，快照保存在`/opt/rke/etcd-snapshots`中。运行该命令时，将创建一个附加容器来获取快照。快照完成后，容器将自动删除。

在v0.2.0之前，RKE保存了证书的备份，名为`pki.bundle.tar.gz`，也保存在`/opt/rke/etcd-snapshots`中。在v0.2.0之前的版本中，系统恢复需要快照和`pki包`文件。

### rke etcd snapshot-save 命令选项

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
|   `--ssh-agent-auth`      |   [使用SSH Agent认证]({{< baseurl >}}/rke/v0.1.x/en/config-options/#ssh-agent) | |
|   `--ignore-docker-version`  | [禁止Docker版本检查]({{< baseurl >}}/rke/v0.1.x/en/config-options/#supported-docker-versions) | |

### 本地一次性快照示例

```bash
rke etcd snapshot-save --config <cluster.yml> --name <snapshot-name>
```

快照将保存在:  `/opt/rke/etcd-snapshots`

### 一次性快照上传到S3示例

> v0.2.0版本可用

```bash
rke etcd snapshot-save --config cluster.yml --name <snapshot-name>  \
--s3 --access-key <S3_ACCESS_KEY> --secret-key <S3_SECRET_KEY> \
--bucket-name <s3-bucket-name> --s3-endpoint <s3.amazonaws.com>
```

快照将保存在:`/opt/rke/etcd-snapshots`，同时也将上传到S3。

## 自动定时快照

To schedule automatic recurring etcd snapshots, you can enable the `etcd-snapshot` service with [extra configuration options the etcd service](#options-for-the-etcd-snapshot-service). `etcd-snapshot` runs in a service container alongside the `etcd` container. By default, the `etcd-snapshot` service takes a snapshot for every node that has the `etcd` role and stores them to local disk in `/opt/rke/etcd-snapshots`. If you set up the [options for S3](#options-for-the-etcd-snapshot-service), the snapshot will also be uploaded to the S3 backend.

Prior to v0.2.0, along with the snapshots, RKE saves a backup of the certificates, i.e. a file named `pki.bundle.tar.gz`, in the same location. The snapshot and pki bundle file are required for the restore process in versions prior to v0.2.0.

When a cluster is launched with the `etcd-snapshot` service enabled, you can view the `etcd-rolling-snapshots` logs to confirm backups are being created automatically.

```bash
docker logs etcd-rolling-snapshots

time="2018-05-04T18:39:16Z" level=info msg="Initializing Rolling Backups" creation=1m0s retention=24h0m0s
time="2018-05-04T18:40:16Z" level=info msg="Created backup" name="2018-05-04T18:40:16Z_etcd" runtime=108.332814ms
time="2018-05-04T18:41:16Z" level=info msg="Created backup" name="2018-05-04T18:41:16Z_etcd" runtime=92.880112ms
time="2018-05-04T18:42:16Z" level=info msg="Created backup" name="2018-05-04T18:42:16Z_etcd" runtime=83.67642ms
time="2018-05-04T18:43:16Z" level=info msg="Created backup" name="2018-05-04T18:43:16Z_etcd" runtime=86.298499ms
```

### Etcd快照服务选项

根据您的RKE版本，用于配置自动定时快照的选项可能有所不同。

> v0.2.0版本可用

|选项|描述| S3特有 |
|---|---| --- |
|**interval_hours**| The duration in hours between recurring backups.  This supercedes the `creation` option and will override it if both are specified.| |
|**retention**| The number of snapshots to retain before rotation. This supercedes the `retention` option and will override it if both are specified.| |
|**bucket_name**| S3 bucket name where backups will be stored| * |
|**access_key**| S3 access key with permission to access the backup bucket.| * |
|**secret_key** |S3 secret key with permission to access the backup bucket.| * |
|**region** |S3 region for the backup bucket. This is optional.| * |
|**endpoint** |S3 regions endpoint for the backup bucket.| * |

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

#### v0.2.0之前

|选项|描述|
|---|---|
|**Snapshot**|By default, the recurring snapshot service is disabled. To enable the service, you need to define it as part of `etcd` and set it to `true`.|
|**Creation**|By default, the snapshot service will take snapshots every 5 minutes (`5m0s`). You can change the time between snapshots as part of the `creation` directive for the `etcd` service.|
|**Retention**|By default, all snapshots are saved for 24 hours (`24h`) before being deleted and purged. You can change how long to store a snapshot as part of the `retention` directive for the `etcd` service.|

```yaml
services:
    etcd:
      snapshot: true
      creation: 5m0s
      retention: 24h
```

## Etcd灾难恢复

If there is a disaster with your Kubernetes cluster, you can use `rke etcd snapshot-restore` to recover your etcd. This command reverts etcd to a specific snapshot. RKE also removes the old `etcd` container before creating a new `etcd` cluster using the snapshot that you have chosen.

>**Warning:** Restoring an etcd snapshot deletes your current etcd cluster and replaces it with a new one. Before you run the `rke etcd snapshot-restore` command, you should back up any important data in your cluster.

### `rke etcd snapshot-restore`选项

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
| `--ssh-agent-auth`      |   [使用SSH Agent认证]({{< baseurl >}}/rke/v0.1.x/en/config-options/#ssh-agent) | |
| `--ignore-docker-version`  | [禁止Docker版本检查]({{< baseurl >}}/rke/v0.1.x/en/config-options/#supported-docker-versions) |

### 从本地快照还原示例

当从本地快照恢复etcd时，假设快照位于`/opt/rke/etcd-snapshots`中。在rke v0.2.0之前的版本中，`pki.bundle.tar.gz`文件也应该在相同的位置, 从v0.2.0开始，不再需要这个文件，因为v0.2.0改变了[Kubernetes集群状态的存储方式]({{< baseurl >}}/rke/v0.1.x/en/installation/# Kubernetes -cluster-state)。

```bash
rke etcd snapshot-restore --config cluster.yml --name mysnapshot
```

### 从S3中还原快照示例

> v0.2.0版本可用

当从S3中的快照恢复etcd时，命令需要S3信息，以便连接到S3后端并检索快照。

```bash
rke etcd snapshot-restore --config cluster.yml --name snapshot-name \
--s3 --access-key S3_ACCESS_KEY --secret-key S3_SECRET_KEY \
--bucket-name s3-bucket-name --s3-endpoint s3.amazonaws.com
```

## 示例

在本例中，Kubernetes集群部署在两个AWS节点上。

|  Name |    IP    |          Role          |
|:-----:|:--------:|:----------------------:|
| node1 | 10.0.0.1 | [controlplane, worker] |
| node2 | 10.0.0.2 | [etcd]                 |

### 备份`etcd`集群

以Kubernetes集群的本地快照为例。从v0.2.0开始，您还可以使用[S3选项](#options-for-rke-etcd-snapshot-save)将此快照直接上传到S3后端。

```bash
rke etcd snapshot-save --name snapshot.db --config cluster.yml
```

![etcd snapshot]({{< baseurl >}}/img/rke/rke-etcd-backup.png)

### 将快照存储在S3

从v0.2.0开始，不再需要这个步骤，因为RKE在运行`rke etcd snapshot-save`命令时，可以通过添加[S3选项](#options-for- RKE -etcd-snapshot-save)自动从S3上传和下载快照。

在`node2`上获取etcd快照之后，我们建议将该备份保存在持久性位置。其中一个选项是保存快照文件和`pki.bundle.tar.gz`文件到S3。

> **注意:** 从v0.2.0开始，恢复过程不再需要**pki.bundle.tar.gz**文件。

```bash
# If you're using an AWS host and have the ability to connect to S3
root@node2:~# s3cmd mb s3://rke-etcd-backup
root@node2:~# s3cmd /opt/rke/etcd-snapshots/snapshot.db /opt/rke/etcd-snapshots/pki.bundle.tar.gz s3://rke-etcd-backup/
```

### 将备份放在新节点上

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

### 通过备份恢复etcd到新节点上

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

将新节点添加到`cluster.yml`之后。运行`rke etcd snapshot-restore`从备份启动`etcd`。快照和`pki.bundle.tar.gz`文件需要预先放在`/opt/rke/etcd-snapshot`目录中。

从v0.2.0开始，如果您想直接从S3检索快照，请添加[S3选项](#options-for-rke-etcd-snapshot-restore)。

> **注意:** 从v0.2.0开始，恢复过程不再需要**pki.bundle.tar.gz**文件。

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

## 故障排除

从**v0.1.9**开始，在恢复成功和失败时都会删除**rke-bundle-cert**容器。要调试任何问题，您需要查看从rke生成的**日志文件**。

在**v0.1.8**以及早期版本中，**rke-bundle-cert**容器是etcd恢复失败后遗留下来的。如果您在还原**etcd快照**时遇到问题，那么在尝试进行另一次还原之前，您可以在每个etcd节点上执行以下操作:

```bash
docker container rm --force rke-bundle-cert
```

当**etcd**的备份或恢复成功时，通常会删除`rke-bundle-cert`容器。无论什么时候出现问题，都会留下`rke-bundle-cert`容器。您可以查看日志或检查容器，看看问题出在哪里。

```bash
docker container logs --follow rke-bundle-cert
docker container inspect rke-bundle-cert
```

需要注意的重要一点是容器的挂载和**pki.bundl .tar.gz**的位置。