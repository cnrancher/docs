---
title: 6 - Helm HA离线升级
weight: 6
---

>**注意:** 如果之前使用RKE Add-on安装的Rancher，请根据[从RKE HA迁移到Helm HA]({{< baseurl >}}/rancher/v2.x/cn/upgrades/migrating-from-rke-add-on/)进行迁移。
>
> 从版本v2.0.8开始，Rancher采用`Helm chart`安装和升级。如果要将升级方法从RKE更改为Helm，请按照此过程操作。

## 一、先决条件

1. **准备离线镜像**

    按照[准备离线镜像]({{< baseurl >}}/rancher/v2.x/cn/installation/air-gap-installation/ha/prepare-private-registry/)方法, 同步新版本镜像到离线镜像仓库。

1. **备份集群**

    [数据备份]({{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/backups/)
    如果在升级期间出现问题，可使用此备份进行恢复。

1. **kubectl**

    安装配置[kubectl]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/kubectl/)，使其可以连接集群。

1. **安装或者升级Helm Server和Helm 客户端**

    如果之前是通过RKE部署的rancher，那首先需要安装Helm Server和Helm 客户端，安装方法参考[安装Helm Server和Helm 客户端]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/helm-install/ "" target="_blank")安装最新版本Helm Server和Helm 客户端

## 二、准备升级文件

> **注意**以下操作需要在安装有`helm`和`kubectl`工具并且可以访问互联网的主机上操作

1. 更新本地`helm repo`缓存。

    ```bash
    helm repo update
    ```

1. 查看本地[helm repo]({{< baseurl >}}/rancher/v2.x/cn/installation/server-tags/#helm-chart-repositories).

    ```bash
    helm repo list

    NAME          	      URL
    stable        	      https://kubernetes-charts.storage.googleapis.com
    rancher-<CHART_REPO>	https://releases.rancher.com/server-charts/<CHART_REPO>
    ```

1. 获取`Rancher Charts`离线包。

    指定安装的版本(比如: `latest`或`stable`或者通过`--version`指定获取的版本)，可通过[版本选择]({{< baseurl >}}/rancher/v2.x/cn/installation/server-tags/)查看版本说明。

    ```plain
    helm fetch rancher-stable/rancher --version v2.2.3
    ```

    >**结果** 默认在当前目录下生成`rancher-vx.x.x.tgz`压缩文件，可通过`-d`指定生成的压缩包路径，比如:`helm fetch rancher-stable/rancher --version v2.2.3 -d /home`，将会在`/home`目录下生成`rancher-vx.x.x.tgz`压缩文件。

## 三、更新 Rancher

拷贝`rancher-vx.x.x.tgz`文件到离线环境中安装有`helm客户端和kubectl客户端`并可以访问内网集群的主机上，解压`rancher-vx.x.x.tgz`得到`rancher`文件夹。

1. 使用`权威认证证书`安装升级

    > **注意** 升级参数应该以安装时设置的参数为准，将安装参数以`--set key=value`的形式附加到升级命令中。

    ```bash
    kubeconfig=xxx.yml

    helm --kubeconfig=$kubeconfig upgrade rancher ./rancher \
        --set hostname=<修改为自己的域名> \
        --set ingress.tls.source=secret \
        --set service.type=ClusterIP \
        --set rancherImage=<离线镜像仓库地址>/rancher/rancher \
        --set busyboxImage=<离线镜像仓库地址>/rancher/busybox
    ```

1. 使用`自签名证书`安装升级

    > **注意** 升级参数应该以安装时设置的参数为准，将安装参数以`--set key=value`的形式附加到升级命令中。

    ```bash
    kubeconfig=xxx.yml

    helm --kubeconfig=$kubeconfig upgrade rancher ./rancher \
        --set hostname=<修改为自己的域名> \
        --set ingress.tls.source=secret \
        --set service.type=ClusterIP \
        --set privateCA=true \
        --set rancherImage=<离线镜像仓库地址>/rancher/rancher \
        --set busyboxImage=<离线镜像仓库地址>/rancher/busybox
    ```

> 因为是离线安装，所以需要指定离线镜像名称。`镜像tag`不需要指定，会自动根据chart版本获取。Rancher Pod中有两个容器，一个是rancher,一个是用于收集审计日志的busybox，更多配置参考[rancher高级设置]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/advanced-settings/)