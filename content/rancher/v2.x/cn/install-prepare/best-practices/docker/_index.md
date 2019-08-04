---
title: 3 - Docker调优
weight: 3
---

1. Docker镜像下载最大并发数

    通过配置镜像上传\下载并发数`max-concurrent-downloads,max-concurrent-uploads`,缩短镜像上传\下载的时间。

1. 配置镜像加速地址

    通过配置镜像加速地址`registry-mirrors`,可以很大程度提高镜像下载速度。

1. 配置Docker存储驱动

    OverlayFS是一个新一代的联合文件系统，类似于AUFS，但速度更快，实现更简单。Docker为OverlayFS提供了两个存储驱动程序:旧版的overlay，新版的[overlay2](https://docs.docker.com/storage/storagedriver/overlayfs-driver/)(更稳定)。

1. 配置日志文件大小

    容器中会产生大量日志文件，很容器占满磁盘空间。通过设置日志文件大小，可以有效控制日志文件对磁盘的占用量。例如：

    ![image-20180910172158993](_index.assets/image-20180910172158993.png)

1. 开启`WARNING: No swap limit support，WARNING: No memory limit support`支持

    对于Ubuntu\Debian系统，执行`docker info`命令时能看到警告`WARNING: No swap limit support或者WARNING: No memory limit support`。因为Ubuntu\Debian系统默认关闭了`swap account或者`功能，这样会导致设置容器内存或者swap资源限制不生效，[解决方法]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/basic-environment-configuration/#3-ubuntu系统-docker-info提示warning-no-swap-limit-support)。

1. (可选)修改Docker默认IP地址

    Docker第一次运行时会自动创建名为docker0的网络接口，默认接口地址为`172.17.0.1/16`。在一些企业中，可能已经使用了这个网段的地址，或者规划以后会使用这个网段的地址。所以，建议在安装好docker服务后，第一时间修改docker0接口地址，避免后期出现网段冲突。

    - 停止docker运行

      `systemctl stop docker.service`

    - 删除已有的docker0接口

      `sudo ip link del docker0`

    - 修改docker配置文件

      在`/etc/docker/daemon.json`中添加`"bip": "169.254.123.1/24",`

1. 综合配置

    ```bash
    touch /etc/docker/daemon.json
    cat > /etc/docker/daemon.json <<EOF
    {
        "oom-score-adjust": -1000,
        "log-driver": "json-file",
        "log-opts": {
        "max-size": "100m",
        "max-file": "3"
        },
        "max-concurrent-downloads": 10,
        "max-concurrent-uploads": 10,
        "bip": "169.254.123.1/24",
        "registry-mirrors": ["https://7bezldxe.mirror.aliyuncs.com"],
        "storage-driver": "overlay2",
        "storage-opts": [
        "overlay2.override_kernel_check=true"
        ]
    }
    EOF
    systemctl daemon-reload && systemctl restart docker
    ```

1. docker.service配置

    对于CentOS系统，docker.service默认位于`/usr/lib/systemd/system/docker.service`；对于Ubuntu系统，docker.service默认位于`/lib/systemd/system/docker.service`。编辑`docker.service`，添加以下参数。

    - 防止docker服务OOM：
    `OOMScoreAdjust=-1000`

    - 开启iptables转发链：
    `ExecStartPost=/usr/sbin/iptables -P FORWARD ACCEPT` (centos)
    ![image-20190615165436722](assets/image-20190615165436722.png)
    `ExecStartPost=/sbin/iptables -P FORWARD ACCEPT` (ubuntu)

    ![image-20190615170819489](assets/image-20190615170819489.png)
