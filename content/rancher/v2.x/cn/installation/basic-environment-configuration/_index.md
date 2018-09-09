---
title: 1 - 基础环境配置
weight: 1
---

{{% accordion id="option-a" label="一、主机配置" %}}

### 1、配置要求

硬件需求根据Rancher部署的规模进行扩展。根据需求配置每个节点。

| 部署大小 | 集群(个)  | 节点(个) | vCPU                                            | 内存 |
| -------- | --------- | -------- |     ------------------------------------------- | ---- |
| 小       | 不超过10  | 最多50   | 2C                                              | 4GB  |
| 中       | 不超过100 | 最多500  | 8C                                              | 32GB |
| 大       | 超过100   | 超过500  | [联系Rancher](https://www.cnrancher.com/contact/) |      |

### 2、操作系统选择

- Ubuntu 16.04(64位)
- Centos/RedHat Linux 7.5+(64位)
- RancherOS 1.3.0+(64位)

> Ubuntu操作系统有Desktop和Server版本，选择安装server版本。

### 3、Docker版本选择

支持的Docker版本

- 1.12.6
- 1.13.1
- 17.03.2

### 4、主机名配置

因为K8S的规定，主机名只支持包含 `-` 和 `.`(中横线和点)两种特殊符号，并且主机名不能出现重复。

### 5、Hosts

配置每台主机的hosts(/etc/hosts),添加`$hostname host_ip`到hosts文件中。

### 6、CentOS关闭selinux

`sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config`

### 7、关闭防火墙(可选)或者放行相应端口

对于刚刚接触Rancher的用户，建议在关闭防火墙的测试环境或桌面虚拟机来运行rancher，以避免出现网络通信问题。

- 关闭防火墙

    1、CentOS

    `systemctl stop firewalld.service && systemctl disable firewalld.service`

    2、Ubuntu

    `ufw disable`

- 端口放行

    端口放行请查看[端口需求]({{< baseurl >}}/rancher/v2.x/cn/installation/references/)

### 8、配置主机时间、时区、系统语言

- 查看时区

    `date -R`或者`timedatectl`

- 修改时区

    ```ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime```

- 修改系统语言环境

    ```sudo echo 'LANG="en_US.UTF-8"' >> /etc/profile;source /etc/profile```

- 配置主机NTP时间同步

### 9、Kernel性能调优

```
cat >> /etc/sysctl.conf<<EOF
net.ipv4.neigh.default.gc_thresh1=4096
net.ipv4.neigh.default.gc_thresh2=6144
net.ipv4.neigh.default.gc_thresh3=8192
EOF
```

>数值根据实际环境自行配置，最后执行`sysctl –p`保存配置。

### 10、ETCD集群容错表

建议在ETCD集群中使用奇数个成员,通过添加额外成员可以获得更高的失败容错。具体详情可以查阅[optimal-cluster-size](https://coreos.com/etcd/docs/latest/v2/admin_guide.html#optimal-cluster-size)。

| 集群大小 | MAJORITY | 失败容错 |
| ------------ | -------- | ----------------- |
| 1            | 1        | 0                 |
| 2            | 2        | 0                 |
| 3            | 2        | **1**             |
| 4            | 3        | 1                 |
| 5            | 3        | **2**             |
| 6            | 4        | 2                 |
| 7            | 4        | **3**             |
| 8            | 5        | 3                 |
| 9            | 5        | **4**             |

{{% /accordion %}}
{{% accordion id="option-b" label="二、Docker安装与配置" %}}

### 1、Docker安装

### Ubuntu

- **Docker-ce**

    ```
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
    **Docker-engine**

    Docker-Engine Docker官方已经不推荐使用，请安装Docker-CE。

### CentOS

- **Docker-ce**

    >因为CentOS的安全限制，通过RKE安装K8S集群时候无法使用`root`账户。所以，建议`CentOS`用户使用非`root`用户来运行docker,不管是`RKE`还是`custom`安装k8s,详情查看[无法为主机配置SSH隧道]({{< baseurl >}}/rancher/v2.x/cn/installation/troubleshooting-ha/ssh-tunneling/)。

    ```
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
    **Docker-engine**

    Docker-Engine Docker官方已经不推荐使用，请安装Docker-CE。

### 2、Docker配置

对于通过systemd来管理服务的系统(比如CentOS7.X、Ubuntu16.X), Docker有两处可以配置参数: 一个是`docker.service`服务配置文件,一个是Docker daemon配置文件daemon.json。

1. `docker.service`

    对于CentOS系统，`docker.service`默认位于`/usr/lib/systemd/system/docker.service`；对于Ubuntu系统，`docker.service`默认位于`/lib/systemd/system/docker.service`

2. `daemon.json`

    daemon.json默认位于`/etc/docker/daemon.json`，如果没有可手动创建，基于systemd管理的系统都是相同的路径。通过修改`daemon.json`来改过Docker配置，也是Docker官方推荐的方法。

>以下说明均基于systemd,并通过`/etc/docker/daemon.json`来修改配置。

### 配置镜像下载和上传并发数

从Docker1.12开始，支持自定义下载和上传镜像的并发数，默认值上传为3个并发，下载为5个并发。通过添加"max-concurrent-downloads"和"max-concurrent-uploads"参数对其修改:

```
"max-concurrent-downloads": 3,
"max-concurrent-uploads": 5
```

### 配置镜像加速地址

Rancher从v1.6.15开始到v2.x.x,Rancher系统相关的所有镜像(包括1.6.x上的K8S镜像)都托管在Dockerhub仓库。Dockerhub节点在国外，国内直接拉取镜像会有些缓慢。为了加速镜像的下载，可以给Docker配置国内的镜像地址。

编辑`/etc/docker/daemon.json`加入以下内容

```
{
"registry-mirrors": ["https://7bezldxe.mirror.aliyuncs.com/","https://IP:PORT/"]
}
```

>可以设置多个`registry-mirrors`地址，以数组形式书写，地址需要添加协议头(https或者http)。

### 配置`insecure-registries`私有仓库

Docker默认只信任TLS加密的仓库地址(https)，所有非https仓库默认无法登陆也无法拉取镜像。`insecure-registries`字面意思为不安全的仓库，通过添加这个参数对非https仓库进行授信。可以设置多个`insecure-registries`地址，以数组形式书写，地址不能添加协议头(http)。

编辑`/etc/docker/daemon.json`加入以下内容:

```
{
"insecure-registries": ["192.168.1.100","IP:PORT"]
}
```

### 配置Docker存储驱动

OverlayFS是一个新一代的联合文件系统，类似于AUFS，但速度更快，实现更简单。Docker为OverlayFS提供了两个存储驱动程序:旧版的`overlay`，新版的`overlay2`(更稳定)。

先决条件:

- `overlay2`: Linux内核版本4.0或更高版本，或使用内核版本3.10.0-514+的RHEL或CentOS。
- `overlay`: 主机Linux内核版本3.18+
- 支持的磁盘文件系统
  - ext4(仅限RHEL 7.1)
  - xfs(RHEL7.2及更高版本)，需要启用d_type=true。
  >具体详情参考 [Docker Use the OverlayFS storage driver](https://docs.docker.com/storage/storagedriver/overlayfs-driver/)

编辑`/etc/docker/daemon.json`加入以下内容

```
{
"storage-driver": "overlay2",
"storage-opts": ["overlay2.override_kernel_check=true"]
}
```

### 配置日志驱动

容器在运行时会产生大量日志文件，很容易占满磁盘空间。通过配置日志驱动来限制文件大小与文件的数量。
>限制单个日志文件为`100M`,最多产生`3`个日志文件

```
{
"log-driver": "json-file",
"log-opts": {
    "max-size": "100m",
    "max-file": "3"
    }
}
```

### 3、Ubuntu系统 ，docker info提示WARNING: No swap limit support

Ubuntu系统下，默认cgroups未开启swap account功能，将会导致需要swap的容器出错。通过修改grub启动参数来开启swap account功能:

```
sudo sed -i 's/GRUB_CMDLINE_LINUX=".*"/GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1 net.ifnames=0"/g'  /etc/default/grub
sudo update-grub
```

> 通过以上命令可自动配置参数，如果`/etc/default/grub`非默认配置，需根据实际参数做调整。

{{% /accordion %}}
{{% accordion id="option-c" label="三、仓库配置" %}}

- 离线安装镜像仓库配置

    在线情况下，Rancher部署kubenetes或者部署其他系统组件时，都是通过dockerhub拉取镜像，Dockerhub上rancher仓库为公开仓库，不用登录即可拉取镜像。如果是在离线环境下安装kubenetes集群,那么对应的项目需要为公开权限，不用登录即可拉取镜像。因为在Rancher2.0全局部署kubenetes时,无法使用Registries功能，对于私有项目无法代理提供登录信息，从而导致无法拉取镜像。

    > **提示** 以上配置完成后，建议重启一次主机。

{{% /accordion %}}
