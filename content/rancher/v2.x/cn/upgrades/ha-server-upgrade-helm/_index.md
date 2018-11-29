---
title: 5 - Helm HA升级
weight: 5
---

>**注意:** 如果之前使用RKE Add-on yaml安装Rancher，请参阅以下文档以进行迁移或升级。
>
>* [从RKE安装迁移]({{< baseurl >}}/rancher/v2.x/cn/upgrades/migrating-from-rke-add-on/)
> 从版本v2.0.8开始，Rancher支持`Helm chart`安装和升级，但仍然支持RKE`安装/升级`。如果要将升级方法从RKE更改为Helm，请按照此过程操作。

## 先决条件

从v2.0.7开始，Rancher引入了System项目，该项目是自动创建的，用于存储Kubernetes需要运行的重要命名空间。在升级到v2.0.7 +期间，Rancher希望从所有项目中取消分配这些名称空间。在开始升级之前，请检查系统命名空间以确保它们未分配以防止集群网络问题。

- **备份Rancher集群**

    [创建快照]({{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/backups/ha-backups/)
    如果在升级期间出现问题，可使用此快照进行恢复。

- **kubectl**

    安装配置[kubectl]({{< baseurl >}}/rancher/v2.x/cn/installation/kubectl/)，使其可以连接集群。

- **安装Helm Server和Helm 客户端**

    根据[安装Helm Server和Helm 客户端]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/air-gap-installation/install-rancher/#二-安装helm-server和helm-客户端)安安装最新版本Helm Server和Helm 客户端

## 升级 Rancher

1. 更新本地helm repo缓存。

    ```bash
    helm repo update
    ```

2. 获取[安装Rancher的存储库名称]({{< baseurl >}}/rancher/v2.x/en/installation/server-tags/#helm-chart-repositories).

    ```bash
    helm repo list

    NAME          	      URL
    stable        	      https://kubernetes-charts.storage.googleapis.com
    rancher-<CHART_REPO>	https://releases.rancher.com/server-charts/<CHART_REPO>
    ```

3. 获取Rancher当前运行版本的配置参数。

    ```bash
    helm get values rancher

    hostname: rancher.my.org
    ```

    > **注意:** 根据您在安装Rancher时选择的[SSL配置选项]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install/helm-rancher/rancher-install/#二-安装证书管理器-可选)，此命令可能会列出更多值。

4. 根据前面步骤中的值将Rancher升级到最新版本。

    - 替换`<CHART_REPO>`为列出的存储库(例如: `latest` or `stable`).
    - 获取上一步中的所有值，并将它们附加到命令`--set key=value`。

    ```bash
    helm upgrade rancher rancher-<CHART_REPO>/rancher --set hostname=rancher.my.org
    ```

>**升级后出现网络问题？**
>
> 请参阅[还原集群网络]({{< baseurl >}}/rancher/v2.x/en/upgrades/upgrades/namespace-migration/#restoring-cluster-networking).

## Rolling Back

Should something go wrong, follow the [HA Rollback]({{< baseurl >}}/rancher/v2.x/en/upgrades/rollbacks/ha-server-rollbacks/) instructions to restore the snapshot you took before you preformed the upgrade.
