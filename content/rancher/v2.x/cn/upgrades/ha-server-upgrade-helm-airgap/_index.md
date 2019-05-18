---
title: 6 - Helm HA离线升级
weight: 6
---

## 一、先决条件

从v2.0.7开始，Rancher引入了System项目，该项目是自动创建的，用于存储Kubernetes需要运行的重要命名空间。在升级到v2.0.7 +期间，Rancher希望从所有项目中取消分配这些命名空间。在开始升级之前，请检查系统命名空间以确保它们未分配以防止集群网络问题。

- **准备离线镜像**

    按照[准备离线镜像]({{< baseurl >}}/rancher/v2.x/cn/installation/air-gap-installation/ha/prepare-private-registry/)方法, 同步新版本镜像到离线镜像仓库。

- **备份集群**

    [创建快照]({{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/backups/ha-backups/)
    如果在升级期间出现问题，可使用此快照进行恢复。

- **kubectl**

    安装配置[kubectl]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/kubectl/)，使其可以连接集群。

- **安装或者升级Helm Server和Helm 客户端**

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

    指定安装的版本(比如: `latest`或`stable`或者通过`--version`指定版本)，可通过[版本选择]({{< baseurl >}}/rancher/v2.x/cn/installation/server-tags/)查看版本说明。

    ```plain
    helm fetch rancher-stable/rancher --version v2.2.3
    ```

    >**结果** 将会在当前目录生成`rancher-vx.x.x.tgz`压缩文件。

1. 拷贝.tgz文件到离线环境中安装有`helm客户端和kubectl客户端`，并可以访问内网集群的主机上，解压.tgz得到`rancher`文件夹；

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

    > 不同的安装方式显示的参数将不相同

1. 根据上一步骤中获取的值，将Rancher升级到最新版本。

    根据上一步骤中的获取的参数值，将它们以`--set key=value`的形式附加到升级命令中；

    ```bash
    helm upgrade rancher ./rancher \
    --set hostname=demo.cnrancher.com \
    --set ingress.tls.source=secret \
    --set service.type=ClusterIP
    --set rancherImage=<离线镜像仓库地址>/rancher/rancher
    --set busyboxImage=<离线镜像仓库地址>/rancher/busybox
    ```

    > 因为是离线安装，所以需要指定离线镜像名称。`镜像tag`不需要指定，会自动根据chart版本获取。Rancher Pod中有两个容器，一个是rancher,一个是用于收集审计日志的busybox，更多配置参考[rancher高级设置]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/advanced-settings/)