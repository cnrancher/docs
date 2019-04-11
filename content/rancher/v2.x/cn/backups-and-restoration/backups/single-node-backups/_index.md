---
title: 1 - 单节点备份
weight: 1
---

在完成Rancher的单节点安装后，或在升级Rancher到新版本之前，需要对Rancher进行数据备份。如果在Rancher数据损坏或者丢失，或者升级遇到问题时，可以通过最新的备份进行数据恢复。

## 一、备份准备

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

## 二、创建备份

1. 浏览器访问Rancher UI，记下浏览器左下角显示的版本号(例如:`v2.0.0`),在后续备份过程中需要这个版本号

2. 停止当前运行Rancher Server的容器,替换`<RANCHER_CONTAINER_ID>`为你真实的Rancher容器的ID

    ```bash
    docker stop `<RANCHER_CONTAINER_ID>`
    ```

    >**提示:** 你可以输入`docker ps`命令获取Rancher容器的ID

3. 创建数据卷容器

    备份当前Rancher Server容器的数据到数据卷容器中

    ```bash
    docker create \
    --volumes-from <RANCHER_CONTAINER_NAME> \
    --name rancher-data-<DATE> \
    rancher/rancher:<RANCHER_CONTAINER_TAG>
    ```

4. 创建Rancher server数据卷容器备份

    在升级期间，新的容器需要链接到数据卷容器，并且会对数据卷容器中的数据进行`更新/更改`。因此，需要提前对数据卷容器进行备份，`以防升级失败时用于数据回滚`。

    ```bash
    docker run  \
    --volumes-from rancher-data-<DATE> \
    -v $PWD:/backup \
    alpine \
    tar zcvf /backup/rancher-data-backup-<RANCHER_VERSION>-<DATE>.tar.gz /var/lib/rancher
    ```

5. 备份完成后可重启Rancher服务容器

6. 了解数据恢复，请点击[单节点数据恢复](../../restorations/single-node-restoration/)
