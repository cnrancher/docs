---
title: 5 - Helm HA升级
weight: 5
---

>**注意:** 如果之前使用RKE Add-on安装的Rancher，请根据[从RKE HA迁移到Helm HA]({{< baseurl >}}/rancher/v2.x/cn/upgrades/migrating-from-rke-add-on/)进行迁移。
>
> 从版本v2.0.8开始，Rancher采用`Helm chart`安装和升级。如果要将升级方法从RKE更改为Helm，请按照此过程操作。

## 一、先决条件

1. **备份Rancher集群**

    如果在升级期间出现问题，可使用[数据备份]({{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/backups/ha-backups/)进行恢复

1. **kubectl**

    安装配置[kubectl]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/kubectl/)，升级将使用kubectl操作。

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

1. 使用`权威认证证书`安装升级

    > **注意** 升级参数应该以安装时设置的参数为准，将安装参数以`--set key=value`的形式附加到升级命令中。

    ```bash
    kubeconfig=xxx.yaml

    helm --kubeconfig=$kubeconfig upgrade \
        rancher rancher-stable/rancher \
        --version v2.2.3 \
        --set hostname=<修改为自己的域名> \
        --set ingress.tls.source=secret \
        --set service.type=ClusterIP \
        --set rancherImage=<离线镜像仓库地址>/rancher/rancher \
        --set busyboxImage=<离线镜像仓库地址>/rancher/busybox
    ```

    >通过`--version`指定升级版本，`镜像tag`不需要指定，会自动根据chart版本获取。

1. 使用`自签名证书`安装升级

    > **注意** 升级参数应该以安装时设置的参数为准，将安装参数以`--set key=value`的形式附加到升级命令中。

    ```bash
    kubeconfig=xxx.yaml

    helm --kubeconfig=$kubeconfig upgrade \
        rancher rancher-stable/rancher \
        --version v2.2.3 \
        --set hostname=<修改为自己的域名> \
        --set ingress.tls.source=secret \
        --set service.type=ClusterIP \
        --set privateCA=true \
        --set rancherImage=<离线镜像仓库地址>/rancher/rancher \
        --set busyboxImage=<离线镜像仓库地址>/rancher/busybox
    ```

    >通过`--version`指定升级版本，`镜像tag`不需要指定，会自动根据chart版本获取。

> 更多配置参考[rancher高级设置]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/advanced-settings/).