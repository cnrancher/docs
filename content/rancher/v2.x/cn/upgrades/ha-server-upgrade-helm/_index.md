---
title: 5 - Helm HA升级
weight: 5
---

>**注意:** 如果之前使用RKE Add-on安装的Rancher，请参阅以下文档以进行迁移或升级。
>
>* [从RKE安装迁移]({{< baseurl >}}/rancher/v2.x/cn/upgrades/migrating-from-rke-add-on/)
> 从版本v2.0.8开始，Rancher采用`Helm chart`安装和升级，但仍然支持RKE`安装/升级`。如果要将升级方法从RKE更改为Helm，请按照此过程操作。

## 一、先决条件

从v2.0.7开始，Rancher引入了System项目，该项目是自动创建的，用于存储Kubernetes需要运行的重要命名空间。在升级到v2.0.7 +期间，Rancher希望从所有项目中取消分配这些命名空间。在开始升级之前，请检查系统命名空间以确保它们未分配以防止集群网络问题。

1. **备份Rancher集群**

    [创建快照]({{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/backups/ha-backups/)
    如果在升级期间出现问题，可使用此快照进行恢复。

1. **kubectl**

    安装配置[kubectl]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/kubectl/)，使其可以连接集群。

1. **安装或者升级Helm Server和Helm 客户端**

    如果之前是通过RKE部署的rancher，那首先需要安装Helm Server和Helm 客户端，安装方法参考[安装Helm Server和Helm 客户端]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/helm-install/ "" target="_blank")安装最新版本Helm Server和Helm 客户端

## 二、升级文件准备

1. 更新本地helm repo缓存；

    ```bash
    helm repo update
    ```

1. 查看本地[helm repo]({{< baseurl >}}/rancher/v2.x/cn/installation/server-tags/#helm-chart-repositories)；

    ```bash
    helm repo list

    NAME          	      URL
    stable        	      https://kubernetes-charts.storage.googleapis.com
    rancher-<CHART_REPO>	https://releases.rancher.com/server-charts/<CHART_REPO>
    ```

## 三、更新 Rancher

1. 获取当前运行Rancher的配置参数；

    ```bash
    helm get values rancher
    ```

    >返回结果示例

    ```plant
    hostname: demo.cnrancher.com
    ingress:
      tls:
        source: secret
    service:
      type: ClusterIP
    ```

    > 不同的安装方式显示的参数不相同

1. 根据上一步骤中获取的值，将Rancher升级到最新版本。

    根据上一步骤中的获取的参数值，将它们以`--set key=value`的形式附加到升级命令中；

    ```bash
    helm upgrade rancher ./rancher \
    --set hostname=demo.cnrancher.com \
    --set ingress.tls.source=secret \
    --set service.type=ClusterIP
    ```

    > 更多配置参考[rancher高级设置]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/advanced-settings/).