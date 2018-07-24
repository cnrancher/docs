---
title: 单节点备份
weight: 25
---

在完成Rancher的单节点安装后，或在升级Rancher到新版本之前，需要对Rancher进行数据备份。如果在Rancher数据损坏或者丢失，或者升级遇到问题时，可以通过最新的备份进行数据恢复。

1. 浏览器访问Rancher UI，记下浏览器左下角显示的版本号(例如：`v2.0.0`）,在后续备份过程中需要这个版本号

2. 停止当前运行Rancher Server的容器,替换`<RANCHER_CONTAINER_ID>`为你真实的Rancher容器的ID

    ```bash
    docker stop `<RANCHER_CONTAINER_ID>`
    ```

    >**提示:** 你可以输入`docker ps`命令获取Rancher容器的ID

3. 创建数据卷容器,备份当前Rancher Server容器运行的数据到数据卷容器中。

    ```bash
    docker create \
    --volumes-from <RANCHER_CONTAINER_ID> \
    --name rancher-backup-<RANCHER_IMAGES_TAG> \
    rancher/rancher:<RANCHER_IMAGES_TAG>
    ```
    - 替换`<RANCHER_CONTAINER_ID>`为上一步获取到的Rancher容器的ID。
    - 替换`<RANCHER_IMAGES_TAG>`为先决条件中获取到的Rancher版本号。

    >在Rancher的版本规划中，Rancher UI左下角显示的版本号始终与镜像的ATG号保持一致，例如: `rancher/rancher:v2.0.4`

4. 以上步骤利用Docker原生的创建数据卷容器来备份数据，Docker会把指定容器中的数据卷中数据全部复制到创建的数据卷容器中，从而实现备份的功能，[了解数据卷容器](https://docs.docker.com/storage/volumes/#backup-restore-or-migrate-data-volumes)。

5. 数据恢复请点击[单节点数据恢复](../../restorations/single-node-restoration/)