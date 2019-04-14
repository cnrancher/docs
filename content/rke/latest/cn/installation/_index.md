---
title: 2 - 安装
weight: 2
---

## 一、下载rke二进制文件

1. 浏览器访问[RKE Releases](https://github.com/rancher/rke/releases/latest)页面,下载符合操作系统的最新RKE安装程序：

    - **MacOS**: `rke_darwin-amd64`
    - **Linux**: `rke_linux-amd64`
    - **Windows**: `rke_windows-amd64.exe`

1. 确保刚下载的RKE二进制文件是可执行文件。打开终端，将目录切换为RKE二进制文件的位置，然后运行以下命令之一。

    >**使用 Windows?**
    >该文件已经是可执行文件，跳过此步骤访问 [准备Kubernetes群集的节点](#二-准备kubernetes集群的节点).

    ```bash
    # MacOS
    chmod +x rke_darwin-amd64
    # Linux
    chmod +x rke_linux-amd64
    ```

1. 通过运行以下命令确认RKE是可执行的：

    ```bash
    # MacOS
    ./rke_darwin-amd64 --version
    # Linux
    ./rke_linux-amd64 --version
    ```

## 二、准备Kubernetes集群的节点

Kubernetes集群组件在Linux系统上以docker容器的形式运行，您可以使用熟悉的Linux发行版，只要它可以满足Docker和Kubernetes的运行需要。

## 三、创建rke配置文件

有两种简单的方法可以创建`cluster.yml`：

- 使用我们的最小值rke配置[cluster.yml]({{< baseurl >}}/rke/latest/cn/example-yamls/#最小-cluster-yml-示例)并根据将使用的节点更新它；
- 使用`rke config`向导式生成配置；

### 1、运行`rke config`

```bash
./rke_darwin-amd64 config
```

#### 指定名称创建配置文件

```bash
rke config --name cluster.yml
```

#### 创建空的`cluster.yml`

如果需要空的`cluster.yml`模板，可以使用该`--empty`参数生成空模板。

```bash
rke config --empty --name cluster.yml
```

#### 打印`cluster.yml`

您可以使用`--print`参数将生成的配置打印到stdout，而不是创建文件。

```bash
rke config --print
```

## 四、RKE部署Kubernetes集群

创建`cluster.yml`完成后，可以使用简单的命令部署群集。此命令假定该`cluster.yml`文件与运行该命令的目录位于同一目录中。

```bash
# MacOS
./rke_darwin-amd64 up
# Linux
./rke_linux-amd64 up
```

在创建Kubernetes集群时会有日志语句。

```bash
./rke_darwin-amd64 up
INFO[0000] Building Kubernetes cluster
INFO[0000] [dialer] Setup tunnel for host [10.0.0.1]
INFO[0000] [network] Deploying port listener containers
INFO[0000] [network] Pulling image [alpine:latest] on host [10.0.0.1]
...
INFO[0101] Finished building Kubernetes cluster successfully
```

当最后一行显示`Finished building Kubernetes cluster successfully`表示群集已部署完成。作为Kubernetes创建过程的一部分，已创建并编写了一个`kubeconfig`文件，通过`kube_config_cluster.yml`文件开始交互您的Kubernetes集群。
