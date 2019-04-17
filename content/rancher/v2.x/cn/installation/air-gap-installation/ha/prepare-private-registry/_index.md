---
title: 2 - 准备离线镜像
weight: 2
aliases:
  - /rancher/v2.x/en/installation/air-gap-installation/prepare-private-reg/
---

## 一、准备文件

Rancher HA安装需要使用来自3个源的镜像，将3个源合并到一个名为`rancher-images.txt`的文件中。

1. 使用可以访问Internet的计算机，访问我们的版本[发布页面](https://github.com/rancher/rancher/releases)，找到需要安装的`Rancher 2.xx`版本。不要下载的版本标示`rc`或者`Pre-release`，因为它们不适用于稳定的生产环境。

    ![Choose Release Version]({{< baseurl >}}/img/rancher/choose-release-version.png)

1. 从发行版的***\*Assets\****部分，下载以下三个文件，这些文件是在离线环境中安装Rancher所必需的：

    | 文件                     | 描述                                                         |
    | :----------------------- | :----------------------------------------------------------- |
    | `rancher-images.txt`     | 此文件包含安装Rancher所需的所有镜像的列表。                  |
    | `rancher-save-images.sh` | 此脚本`rancher-images.txt`从Docker Hub中下载所有镜像并将所有镜像保存为`rancher-images.tar.gz`。 |
    | `rancher-load-images.sh` | 此脚本从`rancher-images.tar.gz`文件加载镜像，并将其推送到您的私有镜像仓库。 |

1. 确保`rancher-save-images.sh`可执行。

    ```bash
    chmod +x rancher-save-images.sh
    ```

1. 通过RKE生成镜像清单

    ```bash
    rke config --system-images >> ./rancher-images.txt
    ```

1. 获取`cert-manager`镜像，[cert-manager](https://github.com/helm/charts/tree/master/stable/cert-manager)版本查询。

    1.  获取最新的`cert-manager` chart并解析模板以获取镜像详细信息。

        ```plain
        helm fetch stable/cert-manager
        helm template ./cert-manager-<version>.tgz | grep -oP '(?<=image: ").*(?=")' >> ./rancher-images.txt
        ```

    2. 对镜像列表进行排序和去重，以消除源之间的镜像重叠。

        ```plain
        sort -u rancher-images.txt -o rancher-images.txt
        ```

1. 使用`rancher-save-images.sh`和`rancher-images.txt`创建所有镜像的压缩包文件。

    ```plain
    ./rancher-save-images.sh --image-list ./rancher-images.txt
    ```

    **结果:** 生成`rancher-images.tar.gz`文件

## 二、同步镜像

1. 在可用访问Internet的主机中，使用`rancher-save-images.sh`和 `rancher-images.txt`创建所有所需镜像的压缩包。

    > **注意:**镜像同步需要接近20GB的空磁盘空间。

1. 如果需要，请登入私有镜像仓库。

     ```plain
    docker login <REGISTRY.YOURDOMAIN.COM:PORT>
    ```

1. 使用`rancher-load-images.sh`加载`rancher-images.tar.gz`，并将镜像推送到私有仓库。

    ```plain
    ./rancher-load-images.sh --image-list ./rancher-images.txt \
    --registry <REGISTRY.YOURDOMAIN.COM:PORT>
    ```
