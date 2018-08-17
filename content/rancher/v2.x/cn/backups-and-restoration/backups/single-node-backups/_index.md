---
title: 1、单节点备份
weight: 1
---

在完成Rancher的单节点安装后，或在升级Rancher到新版本之前，需要对Rancher进行数据备份。如果在Rancher数据损坏或者丢失，或者升级遇到问题时，可以通过最新的备份进行数据恢复。

1. 浏览器访问Rancher UI，记下浏览器左下角显示的版本号(例如:`v2.0.0`),在后续备份过程中需要这个版本号

2. 停止当前运行Rancher Server的容器,替换`<RANCHER_CONTAINER_ID>`为你真实的Rancher容器的ID

    ```bash
    docker stop `<RANCHER_CONTAINER_ID>`
    ```

    >**提示:** 你可以输入`docker ps`命令获取Rancher容器的ID

3. 创建数据卷容器

    备份当前Rancher Server容器的数据到数据卷容器中

    - 替换`<RANCHER_CONTAINER_ID>`为上一步获取到的Rancher容器ID

    - 替换`<RANCHER_IMAGES_TAG>`为第一步获取到的Rancher版本号

    ```bash
    docker create \
    --volumes-from <RANCHER_CONTAINER_ID> \
    --name rancher-backup-<RANCHER_IMAGES_TAG> \
    rancher/rancher:<RANCHER_IMAGES_TAG>
    ```
    >在Rancher的版本规划中，Rancher UI左下角显示的版本号始终与镜像的ATG号保持一致，例如: `rancher/rancher:v2.0.4`

4. 创建Rancher server数据卷容器备份

    在升级期间，新的容器需要链接到数据卷容器，并且会对数据卷容器中的数据进行`更新/更改`。因此，需要提前对数据卷容器进行备份，以防升级失败时用于数据回滚。

    ```bash
    docker run --volumes-from rancher-backup-<RANCHER_IMAGES_TAG> \
    -v $ PWD:/backup \
    alpine \
    tar zcvf /backup/rancher-backup-<RANCHER_IMAGES_TAG>.tar.gz /var/lib/rancher
    ```
5. 备份完成后可重启Rancher服务容器

6. 了解数据恢复，请点击[单节点数据恢复](../../restorations/single-node-restoration/)