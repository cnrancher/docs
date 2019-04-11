---
title: 1 - 单节点恢复
weight: 1
---

## 一、恢复准备

以下信息需要提前准备，在备份时替换相应的值。

| Placeholder                | Example                    | Description                                               |
| -------------------------- | -------------------------- | --------------------------------------------------------- |
| `<RANCHER_CONTAINER_TAG>`  | `v2.0.5`                   | 初始安装Rancher时使用的`rancher/rancher`镜像版本|
| `<RANCHER_CONTAINER_NAME>` | `festive_mestorf`          | Rancher容器名称                       |
| `<RANCHER_VERSION>`        | `v2.0.5`                   |创建的Rancher数据备份对应的Rancher版本|
| `<DATE>`                   | `9-27-18`                  | 备份创建时间   |
<br/>

在终端中输入`docker ps`查询`<RANCHER_CONTAINER_TAG>`和`<RANCHER_CONTAINER_NAME>`

![Placeholder Reference]({{< baseurl >}}/img/rancher/placeholder-ref.png)

## 二、集群恢复

1、停止当前运行的Rancher容器.可通过`docker ps`查看`<RANCHER_CONTAINER_NAME>`

```bash
docker stop <RANCHER_CONTAINER_NAME>
```

2、复制[单节点备份](../../backups/single-node-backups/)的压缩文件(`rancher-data-backup-<RANCHER_VERSION>-<DATE>.tar.gz`)到rancher主机上，通过`cd`命令切换到压缩文件所在的目录，并执行以下命令：

>**警告!** 此命令将从Rancher Server容器中删除所有数据。

```bash
docker run  \
--volumes-from <RANCHER_CONTAINER_NAME> \
-v $PWD:/backup \
alpine \
sh -c "rm /var/lib/rancher/* -rf && tar zxvf /backup/rancher-data-backup-<RANCHER_VERSION>-<DATE>.tar.gz"
```

>**注意** 需要替换`<RANCHER_CONTAINER_NAME>,<RANCHER_VERSION>,<DATE>`

3、重新启动Rancher Server容器

```bash
docker start <RANCHER_CONTAINER_NAME>
```