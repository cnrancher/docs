---
title: 1 - 基础环境配置
weight: 1
---

## 一、主机配置

### 1、配置要求

参考[节点要求](../requirements)

### 2、主机名配置

因为K8S的规定，主机名只支持包含 `-` 和 `.`(中横线和点)两种特殊符号，并且主机名不能出现重复。

### 3、Hosts

配置每台主机的hosts(/etc/hosts),添加`host_ip $hostname`到`/etc/hosts`文件中。

### 4、CentOS关闭selinux

`sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config`

### 5、关闭防火墙(可选)或者放行相应端口

对于刚刚接触Rancher的用户，建议在关闭防火墙的测试环境或桌面虚拟机来运行rancher，以避免出现网络通信问题。

- 关闭防火墙

    1、CentOS

    `systemctl stop firewalld.service && systemctl disable firewalld.service`

    2、Ubuntu

    `ufw disable`

- 端口放行

    端口放行请查看[端口需求]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/references/)

### 6、配置主机时间、时区、系统语言

- 查看时区

    `date -R`或者`timedatectl`

- 修改时区

    ```ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime```

- 修改系统语言环境

    ```sudo echo 'LANG="en_US.UTF-8"' >> /etc/profile;source /etc/profile```

- 配置主机NTP时间同步

### 7、Kernel性能调优

```bash
cat >> /etc/sysctl.conf<<EOF
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-iptables=1
net.ipv4.neigh.default.gc_thresh1=4096
net.ipv4.neigh.default.gc_thresh2=6144
net.ipv4.neigh.default.gc_thresh3=8192
EOF
```

>数值根据实际环境自行配置，最后执行`sysctl -p`保存配置。

### 8、内核模块

>**警告** 如果要使用ceph存储相关功能，需保证worker节点加载`RBD模块`

以下模块需要在主机上加载

| 模块名称               |
| ---------------------- |
| br_netfilter           |
| ip6_udp_tunnel         |
| ip_set                 |
| ip_set_hash_ip         |
| ip_set_hash_net        |
| iptable_filter         |
| iptable_nat            |
| iptable_mangle         |
| iptable_raw            |
| nf_conntrack_netlink   |
| nf_conntrack           |
| nf_conntrack_ipv4      |
| nf_defrag_ipv4         |
| nf_nat                 |
| nf_nat_ipv4            |
| nf_nat_masquerade_ipv4 |
| nfnetlink              |
| udp_tunnel             |
| VETH                   |
| VXLAN                  |
| x_tables               |
| xt_addrtype            |
| xt_conntrack           |
| xt_comment             |
| xt_mark                |
| xt_multiport           |
| xt_nat                 |
| xt_recent              |
| xt_set                 |
| xt_statistic           |
| xt_tcpudp              |

>模块查询: lsmod | grep <模块名> \
模块加载: modprobe <模块名>\

### 9、ETCD集群容错表

建议在ETCD集群中使用奇数个成员,通过添加额外成员可以获得更高的失败容错。具体详情可以查阅[optimal-cluster-size](https://coreos.com/etcd/docs/latest/v2/admin_guide.html#optimal-cluster-size)。

| 集群大小 | MAJORITY | 失败容错 |
| ------------ | -------- | ----------------- |
| 1            | 1        | **0**                 |
| 2            | 2        | **0**                 |
| 3            | 2        | **1**             |
| 4            | 3        | **1**                 |
| 5            | 3        | **2**             |
| 6            | 4        | **2**               |
| 7            | 4        | **3**             |
| 8            | 5        | **3**                 |
| 9            | 5        | **4**             |

## 二、Docker安装与配置

### 1、Docker安装

### Ubuntu 16.x

- **修改系统源**

    ```bash
    sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
    cat > /etc/apt/sources.list << EOF

    deb http://mirrors.aliyun.com/ubuntu/ xenial main
    deb-src http://mirrors.aliyun.com/ubuntu/ xenial main
    deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main
    deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main
    deb http://mirrors.aliyun.com/ubuntu/ xenial universe
    deb-src http://mirrors.aliyun.com/ubuntu/ xenial universe
    deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
    deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
    deb http://mirrors.aliyun.com/ubuntu/ xenial-security main
    deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main
    deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
    deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security universe

    EOF
    ```

- **Docker-ce安装**

    ```bash
    # 定义安装版本
    export docker_version=18.06.3;
    # step 1: 安装必要的一些系统工具
    sudo apt-get remove docker docker-engine docker.io containerd runc -y;
    sudo apt-get update;
    sudo apt-get -y install apt-transport-https ca-certificates \
        curl software-properties-common bash-completion  gnupg-agent;
    # step 2: 安装GPG证书
    sudo curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | \
        sudo apt-key add -;
    # Step 3: 写入软件源信息
    sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu \
        $(lsb_release -cs) stable";
    # Step 4: 更新并安装 Docker-CE
    sudo apt-get -y update;
    version=$(apt-cache madison docker-ce|grep ${docker_version}|awk '{print $3}');
    # --allow-downgrades 允许降级安装
    sudo apt-get -y install docker-ce=${version} --allow-downgrades;
    # 设置开机启动
    sudo systemctl enable docker;
    ```

    **Docker-engine**

    Docker-Engine Docker官方已经不推荐使用，请安装Docker-CE。

### CentOS 7.x

- **Docker-ce安装**

    >因为CentOS的安全限制，通过RKE安装K8S集群时候无法使用`root`账户。所以，建议`CentOS`用户使用非`root`用户来运行docker,不管是`RKE`还是`custom`安装k8s,详情查看[无法为主机配置SSH隧道]({{< baseurl >}}/rancher/v2.x/cn/faq/troubleshooting-ha/ssh-tunneling/)。

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
    export docker_version=18.06.3
    # step 1: 安装必要的一些系统工具
    sudo yum remove docker docker-client docker-client-latest \
        docker-common docker-latest docker-latest-logrotate \
        docker-logrotate docker-engine -y;
    sudo yum update -y;
    sudo yum install -y yum-utils device-mapper-persistent-data \
        lvm2 bash-completion;
    # Step 2: 添加软件源信息
    sudo yum-config-manager --add-repo \
        http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo;
    # Step 3: 更新并安装 Docker-CE
    sudo yum makecache all;
    version=$(yum list docker-ce.x86_64 --showduplicates | sort -r|grep ${docker_version}|awk '{print $2}');
    sudo yum -y install --setopt=obsoletes=0 docker-ce-${version} docker-ce-selinux-${version};
    # 如果已经安装高版本Docker,可进行降级安装(可选)
    yum downgrade --setopt=obsoletes=0 -y docker-ce-${version} docker-ce-selinux-${version};
    # 把当前用户加入docker组
    sudo usermod -aG docker `<new_user>`;
    # 设置开机启动
    sudo systemctl enable docker;
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

```bash
"max-concurrent-downloads": 3,
"max-concurrent-uploads": 5
```

### 配置镜像加速地址

Rancher从v1.6.15开始到v2.x.x,Rancher系统相关的所有镜像(包括1.6.x上的K8S镜像)都托管在Dockerhub仓库。Dockerhub节点在国外，国内直接拉取镜像会有些缓慢。为了加速镜像的下载，可以给Docker配置国内的镜像地址。

编辑`/etc/docker/daemon.json`加入以下内容

```json
{
"registry-mirrors": ["https://7bezldxe.mirror.aliyuncs.com/","https://IP:PORT/"]
}
```

>可以设置多个`registry-mirrors`地址，以数组形式书写，地址需要添加协议头(https或者http)。

### 配置`insecure-registries`私有仓库

Docker默认只信任TLS加密的仓库地址(https)，所有非https仓库默认无法登陆也无法拉取镜像。`insecure-registries`字面意思为不安全的仓库，通过添加这个参数对非https仓库进行授信。可以设置多个`insecure-registries`地址，以数组形式书写，地址不能添加协议头(http)。

编辑`/etc/docker/daemon.json`加入以下内容:

```json
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

```json
{
"storage-driver": "overlay2",
"storage-opts": ["overlay2.override_kernel_check=true"]
}
```

### 配置日志驱动

容器在运行时会产生大量日志文件，很容易占满磁盘空间。通过配置日志驱动来限制文件大小与文件的数量。
>限制单个日志文件为`100M`,最多产生`3`个日志文件

```json
{
"log-driver": "json-file",
"log-opts": {
    "max-size": "100m",
    "max-file": "3"
    }
}
```

### 3、Ubuntu\Debian系统 ，docker info提示WARNING: No swap limit support

Ubuntu\Debian系统下，默认cgroups未开启swap account功能，这样会导致设置容器内存或者swap资源限制不生效。可以通过以下命令解决:

```bash
# 统一网卡名称为ethx
sudo sed -i 's/en[[:alnum:]]*/eth0/g' /etc/network/interfaces;
sudo sed -i 's/GRUB_CMDLINE_LINUX="\(.*\)"/GRUB_CMDLINE_LINUX="net.ifnames=0 cgroup_enable=memory swapaccount=1 biosdevname=0 \1"/g' /etc/default/grub;
sudo update-grub;
```

> **注意** 通过以上命令可自动配置参数，如果`/etc/default/grub`非默认配置，需根据实际参数做调整。
> **提示** 以上配置完成后，建议重启一次主机。
