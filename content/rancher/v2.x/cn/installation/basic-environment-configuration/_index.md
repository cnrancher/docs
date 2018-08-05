---
title: 基础环境配置
weight: 1
---

## 一、主机配置

### 1、主机名配置

因为K8S的规定，主机名只支持包含 `-` 和 `.`(中横线和点)两种特殊符号。

### 2、hosts

配置每台主机的hosts(/etc/hosts),添加`$hostname host_ip`到hosts文件中。

### 3、CentOS关闭selinux

`sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config`

### 4、关闭防火墙(可选)或者放行相应端口

- 关闭防火墙

1. CentOS

    `systemctl stop firewalld.service && systemctl disable firewalld.service`

2. Ubuntu

    `ufw disable`

- 端口放行

    端口放行请查看[端口需求](/docs/rancher/v2.x/cn/installation/references/)

## 二、Docker安装与配置

### 1、Docker安装

- Ubuntu

    ```bash
    # 定义安装版本
    export docker_version=17.03.2
    # step 1: 安装必要的一些系统工具
    sudo apt-get update
    sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
    # step 2: 安装GPG证书
    sudo curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
    # Step 3: 写入软件源信息
    sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
    # Step 4: 更新并安装 Docker-CE
    sudo apt-get -y update
    version=$(apt-cache madison docker-ce|grep ${docker_version}|awk '{print $3}')
    # --allow-downgrades 允许降级安装
    sudo apt-get -y install docker-ce=${version} --allow-downgrades
    # 设置开机启动
    sudo systemctl enable docker
    ```

- CentOS

    > 因为CentOS的安全限制，通过RKE安装K8S集群时候无法使用`root`账户。所以，建议`CentOS`用户使用非`root`用户来运行docker,不管是`RKE`还是`custom`安装k8s,详情查看[无法为主机配置SSH隧道](/docs/rancher/v2.x/cn/installation/troubleshooting-ha/ssh-tunneling/)。

    ```bash
    # 添加用户(可选)
    sudo adduser `<new_user>`
    # 为新用户设置密码
    sudo passwd `<new_user>`
    # 为新用户添加sudo权限
    sudo echo '<new_user> ALL=(ALL) ALL' >> /etc/sudoers
    # 卸载旧版本Docker软件
    sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  container*
    # 定义安装版本
    export docker_version=17.03.2
    # step 1: 安装必要的一些系统工具
    sudo yum update -y
    sudo yum install -y yum-utils device-mapper-persistent-data lvm2
    # Step 2: 添加软件源信息
    sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    # Step 3: 更新并安装 Docker-CE
    sudo yum makecache all
    version=$(yum list docker-ce.x86_64 --showduplicates | sort -r|grep ${docker_version}|awk '{print $2}')
    sudo yum -y install --setopt=obsoletes=0 docker-ce-${version} docker-ce-selinux-${version}
    # 如果已经安装高版本Docker,可进行降级安装(可选)
    yum downgrade --setopt=obsoletes=0 -y docker-ce-${version} docker-ce-selinux-${version}
    # 把当前用户加入docker组
    sudo usermod -aG docker `<new_user>`
    # 设置开机启动
    sudo systemctl enable docker
    ```

### 2、Docker配置

对于通过systemd来管理服务的系统(比如CentOS7.X、Ubuntu16.X), Docker有两处可以配置参数: 一个是`docker.service`服务配置文件,一个是Docker daemon配置文件daemon.json。

1. docker.service

    对于CentOS系统，`docker.service`默认位于`/usr/lib/systemd/system/docker.service`；对于Ubuntu系统，`docker.service`默认位于`/lib/systemd/system/docker.service`

2. daemon.json

    daemon.json默认位于`/etc/docker/daemon.json`，如果没有可手动创建，基于systemd管理的系统都是相同的路径。通过修改`daemon.json`来改过Docker配置，也是Docker官方推荐的方法。

> 以下说明均基于systemd,并通过`/etc/docker/daemon.json`来修改配置。

- 配置镜像下载和上传并发数

    从Docker1.12开始，支持自定义下载和上传镜像的并发数，默认值上传为3个并发，下载为5个并发。通过添加"max-concurrent-downloads"和"max-concurrent-uploads"参数对其修改：

    ```json
    "max-concurrent-downloads": 3,
    "max-concurrent-uploads": 5
    ```

- 配置镜像加速地址

    Rancher从v1.6.15开始到v2.x.x,Rancher系统相关的所有镜像(包括1.6.x上的K8S镜像)都托管在Dockerhub仓库。Dockerhub节点在国外，国内直接拉取镜像会有些缓慢。为了加速镜像的下载，可以给Docker配置国内的镜像地址。

    编辑`/etc/docker/daemon.json`加入以下内容

    ```json
    {
    "registry-mirrors": ["https://7bezldxe.mirror.aliyuncs.com/","https://IP:PORT/"]
    }
    ```
    > 可以设置多个`registry-mirrors`地址，以数组形式书写，地址需要添加协议头(https或者http)。

- 配置`insecure-registries`私有仓库

    Docker默认只信任TLS加密的仓库地址(https)，所有非https仓库默认无法登陆也无法拉取镜像。`insecure-registries`字面意思为不安全的仓库，通过添加这个参数对非https仓库进行授信。可以设置多个`insecure-registries`地址，以数组形式书写，地址不能添加协议头(http)。

    编辑`/etc/docker/daemon.json`加入以下内容

    ```json
    {
    "insecure-registries": ["192.168.1.100","IP:PORT"]
    }
    ```

- 配置Docker存储驱动

    编辑`/etc/docker/daemon.json`加入以下内容

    ```json
    {
    "storage-driver": "overlay2",
    "storage-opts": ["overlay2.override_kernel_check=true"]
    }
    ```
- Ubuntu系统 ，docker info提示WARNING: No swap limit support

    Ubuntu系统下，默认cgroups未开启swap account功能，将会导致需要swap的容器出错。通过修改grub启动参数来开启swap account功能：

    ```bash
    sudo sed -i 's/GRUB_CMDLINE_LINUX=".*"/GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1 net.ifnames=0"/g'  /etc/default/grub
    sudo update-grub
    ```
> 通过以上命令可自动配置参数，如果`/etc/default/grub`非默认配置，需根据实际参数做调整。

## 三、仓库配置

- 离线安装镜像仓库配置

    在线情况下，Rancher部署kubenetes或者部署其他系统组件时，都是通过dockerhub拉取镜像，Dockerhub上rancher仓库为公开仓库，不用登录即可拉取镜像。如果是在离线环境下安装kubenetes集群,那么对应的项目需要为公开权限，不用登录即可拉取镜像。因为在Rancher2.0全局部署kubenetes时,无法使用Registries功能，对于私有项目无法代理提供登录信息，从而导致无法拉取镜像。
