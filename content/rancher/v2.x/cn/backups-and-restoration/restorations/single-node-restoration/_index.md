---
title: 1 - 单节点恢复
weight: 1
---

如果Rancher数据损坏、丢失，或在升级时遇到问题，可通过备份的数据进行恢复。

1. 停止当前运行的Rancher容器.可通过`docker ps`查看`<RANCHER_CONTAINER_ID>`

    ```
    docker stop <RANCHER_CONTAINER_ID>
    ```

2. 通过创建的数据卷容器`rancher-backup-<RANCHER_IMAGES_TAG>`，创建新的Rancher容器:

    有关数据备份方法，请查阅[单节点备份](../../backups/single-node-backups/).

    ```
    docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    --name rancher-server-`date +%Y%m%d%H%M%S` \
    --volumes-from rancher-backup-<RANCHER_IMAGES_TAG>  \
    rancher/rancher:<RANCHER_IMAGES_TAG>
    ```
    >注: 设置容器名，可以快速定位容器，`date +%Y%m%d%H%M%S` 会精确到秒，防止容器名冲突。