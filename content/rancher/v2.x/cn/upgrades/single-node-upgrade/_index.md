---
title: 1 - 单节点升级
weight: 1
---

>**先决条件:** 打开Rancher web并记下浏览器左下方显示的版本号(例如:`v2.0.0`) ，在升级过程中你需要此版本号

1. 运行以下命令，停止当前运行Rancher Server的容器

      ```
      docker stop <RANCHER_CONTAINER_ID>
      ```
      >**提示:** 你可以输入`docker ps`命令获取Rancher容器的ID

2. 创建当前Rancher Server容器的数据卷容器，以便在升级Rancher Server中使用，命名为rancher-data容器。

    - 替换`<RANCHER_CONTAINER_ID>`为上一步中的相同容器ID。
    - 替换`<RANCHER_CONTAINER_TAG>`为你当前正在运行的Rancher版本，如上面的先决条件中所述。

    ```
    docker create --volumes-from <RANCHER_CONTAINER_ID> \
    --name rancher-data rancher/rancher:<RANCHER_CONTAINER_TAG>
    ```

3. 备份`rancher-data`数据卷容器

    如果升级失败，可以通过此备份还原Rancher Server，容器命名:`rancher-data-snapshot-<CURRENT_VERSION>`.

    - 替换`<RANCHER_CONTAINER_ID>`为上一步中的相同ID。
    - 替换`<CURRENT_VERSION>`为当前安装的Rancher版本的标记。
    - 替换`<RANCHER_CONTAINER_TAG>`为当前正在运行的Rancher版本，如先决条件中所述 。

    ```
    docker create --volumes-from <RANCHER_CONTAINER_ID> \
    --name rancher-data-snapshot-<CURRENT_VERSION> rancher/rancher:<RANCHER_CONTAINER_TAG>
    ```
4. 拉取Rancher的最新镜像。

      ```
      docker pull rancher/rancher:latest
      ```
    >**注意** 如果你正在进行[离线升级]({{< baseurl >}}/rancher/v2.x/cn/upgrades/air-gap-upgrade/)，请在运行docker run命令时将你的私有镜像仓库URL添加到镜像名中
    >
    >例如: `<registry.yourdomain.com:port>/rancher/rancher:latest`

5. 通过`rancher-data`数据卷容器启动新的`Rancher Server`容器。

    ```
    docker run -d --volumes-from rancher-data --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:latest
    ```

    >**注意:** 升级过程会需要一定时间，不要在升级过程中终止升级，强制终止可能会导致数据库迁移错误。
    >
    >升级Rancher Server后，server容器中的数据会保存到`rancher-data`容器中，以便将来升级。

6. 删除旧版本`Rancher Server`容器

    如果你只是停止以前的Rancher Server容器(并且不删除它),则旧版本容器可能随着主机重启后自动运行，导致容器端口冲突。

7. 登录rancher，通过检查浏览器左下角显示的版本，确认是否升级成功。

    >**注意:** 如果升级未成功完成，则可以将Rancher Server及其数据恢复到上一个健康状态。有关更多信息，请查阅[单节点恢复]({{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/restorations/single-node-restoration/)。