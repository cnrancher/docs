---
title: 离线安装
weight: 5
---

Rancher支持从私有镜像仓库进行安装。在每个发行版中，我们都会提供所需的Docker镜像清单和脚本，通过这些脚本你可以把镜像同步到你的私有仓库中。当主机添加到集群时，或启用CICD或启用日志收集功能时，将使用这些Docker镜像。

>**先决条件:** 假设你有自己的私有镜像仓库或其他方式将镜像分发到你的主机。如果你在创建私有镜像仓库方面需要帮助, 请参考： [Docker documentation for private registries](https://docs.docker.com/registry/).
>
>在Rancher v2.0.0中，从私有仓库安装不支持使用身份验证的私有仓库，仓库需要为公开。

## Release文件

- **rancher-images.txt**: 包含该版本所需的所有镜像；
- **rancher-save-images.sh**: 该脚本将从**DockerHub中**拉取所有需要的镜像，并将所有镜像保存为一个名为压缩文件`rancher-images.tar.gz`。拷贝`rancher-images.tar.gz`文件到可以访问内部私有镜像仓库的主机上；
- **rancher-load-images.sh**: 这个脚本会从rancher-images.tar.gz导入镜像，并将它们推送到私有仓库。您必须添加私有仓库地址作为脚本的第一个参数。`rancher-load-images.sh registry.yourdomain.com:5000`;

### 我们将介绍两种离线安装方案：

- **方案1**：一台可访问DockerHub的主机来拉取和保存镜像，另一台可以访问私有仓库的主机来推送镜像；
- **方案2**：有一台可以同时访问DockerHub和私有仓库的主机；

#### **方案1**：一台可访问DockerHub的主机来拉取和保存镜像，另一台可以访问私有仓库的主机来推送镜像；

![Scenario1]({{< baseurl >}}/img/rancher/airgap/privateregistry.svg)

1. 根据你选择的版本(i.e. `https://github.com/rancher/rancher/releases/tag/v2.0.0`) 浏览release页面，并下载 `rancher-save-images.sh` 和 `rancher-load-images.sh` 两个脚本；

2. 复制 `rancher-save-images.sh` 脚本到能访问DockerHub的主机上运行，将需要大致20G磁盘空间；

3. 复制`rancher-images.tar.gz` 和`rancher-load-images.sh`到可以访问私有仓库的主机上，两个文件放在同一级目录下，并运行`rancher-load-images.sh`；

4. 使用[单节点安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/single-node-install).中的说明完成Rancher的安装；

    >**注意:**
    >在进行单节点安装，运行`docker run`命令时，需要将私有仓库地址添加到镜像中。
    > 例如:
    > ```bash
    >docker run -d --restart=unless-stopped \
    >-p 80:80 -p 443:443 \
    > <registry.yourdomain.com:port>/rancher/rancher:latest
    > ```

#### 方案2：有一台可以同时访问DockerHub和私有仓库的主机

![Scenario2]({{< baseurl >}}/img/rancher/airgap/privateregistrypushpull.svg)

1. 保存以下示例脚本，假设命名为：mirror-rancher-images.sh

    ```bash
    #!/bin/sh
     version=v2.0.0
     privateregistry=$1
     IMAGES=`curl -s -
    https://github.com/rancher/rancher/releases/download/$version/rancher-images.txt`
     for IMAGE in $IMAGES; do
        until docker inspect $IMAGE > /dev/null 2>&1; do
          docker pull $IMAGE
        done
        docker tag $IMAGE $privateregistry/$IMAGE
        docker push $privateregistry/$IMAGE
     done
    ```

2. 执行脚本时以参数的形式把私有仓库地址传递给脚本，

    ```bash
    bash mirror-rancher-images.sh <registry.yourdomain.com:port>
    ```

3. 使用[单节点安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/single-node-install).中的说明完成Rancher的安装；

    >**注意: **在进行单节点安装，运行`docker run`命令时，需要将私有仓库地址添加到镜像中。例如:
    > ```bash
    >docker run -d --restart=unless-stopped \
    > -p 80:80 -p 443:443 \
    > <registry.yourdomain.com:port>/rancher/rancher:latest
    > ```
  
### 配置Rancher 使用私有仓库

Rancher需要配置为使用私有注册表作为所需映像的源代码

1. 进入设置视图

     ![Settings]({{< baseurl >}}/img/rancher/airgap/settings.png)

2. 查找`system-default-registry`并**编辑**。 

     ![Edit]({{< baseurl >}}/img/rancher/airgap/edit-system-default-registry.png)

3. 将值改为你的私有仓库地址, e.g. `registry.yourdomain.com:port`. 不要添加 `http://` or `https://`前缀
     ![Save]({{< baseurl >}}/img/rancher/airgap/enter-system-default-registry.png)

    >**注意:** 如果要在启动rancher/rancher容器时配置``system-default-registry`，可以使用环境变量
    >`CATTLE_SYSTEM_DEFAULT_REGISTRY`
    >
    >```bash
    >#!/bin/sh
    >docker run -d -p 80:80 -p 443:443 
    >-e CATTLE_SYSTEM_DEFAULT_REGISTRY=<registry.yourdomain.com:port> 
    ><registry.yourdomain.com:port>/>rancher/rancher:v2.0.0
    >```