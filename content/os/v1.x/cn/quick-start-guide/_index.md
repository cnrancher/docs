---
title: 2 - 入门
weight: 2
---

如果您有特定的RanchersOS机器的要求，请查看我们的[RancherOS运行指南]({{< baseurl >}}/os/v1.x/en/installation/running-rancheros/)。另外，我们将使用[docker-machine]({{< baseurl >}}/os/v1.x/en/installation/running-rancheros/workstation/docker-machine/)启动RancherOS，并向您展示RancherOS的一些功能。

## 使用Docker Machine启动RancherOS

在继续之前，你需要安装[Docker Machine](https://docs.docker.com/machine/)和[VirtualBox](https://www.virtualbox.org/wiki/Downloads) 一旦安装了`VirtualBox和Docker Machine`，就可以运行一条命令让RancherOS运行。

### 创建虚拟机并运行RancherOS

```bash
$ docker-machine create -d virtualbox \
        --virtualbox-boot2docker-url https://releases.rancher.com/os/latest/rancheros.iso \
        --virtualbox-memory 2048 \
        <MACHINE-NAME>
```

### 查看创建的虚拟机

```bash
docker-machine ls
```

### 登录RancherOS

```bash
docker-machine ssh <MACHINE-NAME>
```

## 了解 RancherOS

RancherOS中运行着两个`Docker守护进程`。第一个名为**System Docker**， RancherOS在这里运行`ntpd和syslog`等系统服务。您可以使用`system-docker`命令来控制**System Docker**守护进程。

系统上运行的另一个Docker守护进程是**Docker**，可以使用普通的`docker`命令访问它。

当您第一次启动RancherOS时，Docker守护进程中没有运行容器。但是，如果对系统Docker运行相同的命令，您将看到RancherOS附带的许多系统服务。

> **注意:** `system-docker`需要root权限,所以需要随时使用`sudo`命令。

```bash
$ sudo system-docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS               NAMES
6f56057cf5ba        rancher/os-base:v0.5.0      "/usr/sbin/entry.sh /"   16 seconds ago      Up 15 seconds                           docker
bd5376830237        rancher/os-console:v0.5.0   "/usr/sbin/entry.sh /"   16 seconds ago      Up 15 seconds                           console
ede8ce39fff5        rancher/os-base:v0.5.0      "/usr/sbin/entry.sh n"   16 seconds ago      Up 15 seconds                           network
9e5d18bca391        rancher/os-base:v0.5.0      "/usr/sbin/entry.sh n"   17 seconds ago      Up 16 seconds                           ntp
393b9fb7e30a        rancher/os-udev:v0.5.0      "/usr/sbin/entry.sh /"   18 seconds ago      Up 16 seconds                           udev
dc2cafca3c69        rancher/os-syslog:v0.5.0    "/usr/sbin/entry.sh /"   18 seconds ago      Up 17 seconds                           syslog
439d5535fbfa        rancher/os-base:v0.5.0      "/usr/sbin/entry.sh /"   18 seconds ago      Up 17 seconds                           acpid
```

一些容器在引导时运行，而其他容器，如`console、docker`等，则始终运行。

## 使用 RancherOS

### 部署第一个Docker容器

让我们尝试在Docker守护进程上部署一个普通的Docker容器。RancherOS Docker守护进程与任何其他Docker环境都是相同的，因此所有正常的Docker命令都可以工作。

```bash
docker run -d nginx
```

您可以看到nginx容器已经启动并运行:

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
e99c2c4b8b30        nginx               "nginx -g 'daemon off"   12 seconds ago      Up 11 seconds       80/tcp, 443/tcp     drunk_ptolemy
```

### 部署系统服务容器

下面是一个简单的Docker容器，用于设置`Linux-dash`，这是一个用于监视Linux服务器的最小的低开销web仪表板。Dockerfile是这样的:

```bash
FROM hwestphal/nodebox
MAINTAINER hussein.galal.ahmed.11@gmail.com

RUN opkg-install unzip
RUN curl -k -L -o master.zip https://github.com/afaqurk/linux-dash/archive/master.zip
RUN unzip master.zip
WORKDIR linux-dash-master
RUN npm install

ENTRYPOINT ["node","server"]
```

使用`hwestphal/nodebox`镜像，该镜像使用Busybox镜像并安装`node.js和npm`。我们下载了Linux-dash的源代码，然后运行服务器。默认情况下，Linux-dash将在端口80上运行。

要在系统Docker中运行此容器，请使用以下命令:

```bash
sudo system-docker run -d --net=host --name busydash husseingalal/busydash
```

在命令中，我们使用`——net=host`来告诉System Docker不要将容器的网络装入容器，而是使用主机的网络。运行容器后，您可以通过访问`http://<IP_OF_MACHINE>`来查看监视服务器。

![System Docker Container]({{< baseurl >}}/img/os/Rancher_busydash.png)

要使容器在重新引导期间自动运行，可以创建`/opt/rancher/bin/start`脚本，并添加Docker启动项，以便每次启动时启动Docker容器。

```bash
sudo mkdir -p /opt/rancher/bin
echo "sudo system-docker start busydash" | sudo tee -a /opt/rancher/bin/start.sh
sudo chmod 755 /opt/rancher/bin/start.sh
```

## 使用 ROS

RancherOS可以使用的另一个有用的命令是`ros`，它可以用来控制和配置系统。

```bash
$ sudo ros -v
ros version 0.0.1
```

RancherOS状态由一个云配置文件控制。使用`ros`编辑系统配置，例如查看系统dns配置:

```bash
sudo ros config get rancher.network.dns.nameservers

- 8.8.8.8
- 8.8.4.4
```

当使用本地Busybox控制台时，对控制台的任何更改将在重新引导后丢失，只有对`/home或/opt`的更改将是持久的。您可以使用`ros console switch`切换到[持久控制台]({{< baseurl >}}/os/v1.x/en/installation/custom-builds/custom-console/#console-persistence)并替换本地Busybox控制台。例如，切换到Ubuntu控制台:

```bash
sudo ros console switch ubuntu
```

## 结论

RancherOS是一个简单的Linux发行版，是运行Docker的理想选择。通过采用系统服务的容器化并利用Docker进行管理，RancherOS希望为运行容器提供一个非常可靠、易于管理的操作系统。