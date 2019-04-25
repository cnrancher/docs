---
title: 2 - 安装
weight: 2
---

## 一、rke安装

### 1、二进制文件安装

1. 浏览器访问[RKE Releases](https://github.com/rancher/rke/releases/latest)页面,下载符合操作系统的最新RKE安装程序：

    - **MacOS**：`rke_darwin-amd64`
    - **Linux（Intel / AMD）**：`rke_linux-amd64`
    - **Linux（ARM 32位）**：`rke_linux-arm`
    - **Linux（ARM 64位）**：`rke_linux-arm64`
    - **Windows（32位）**：`rke_windows-386.exe`
    - **Windows（64位）**：`rke_windows-amd64.exe`

1. 运行以下命令给与二进制文件执行权限；

    >**使用 Windows?**
    >该文件已经是可执行文件，跳过此步骤访问[准备Kubernetes集群的节点](#二-准备kubernetes集群的节点).

    ```bash
    # MacOS
    chmod +x rke_darwin-amd64
    # Linux
    chmod +x rke_linux-amd64
    ```

1. 确认RKE可执行：

    ```bash
    # MacOS
    ./rke_darwin-amd64 --version
    # Linux
    ./rke_linux-amd64 --version
    ```

### 2、macos brew安装

1. 安装Homebrew，请参阅<https://brew.sh/>；
1. 通过在终端窗口中运行以下命令来安装RKE：

    ```bash
    brew install rke
    ```

1. 如果您已经通过brew安装了`RKE` ，则可以通过运行以下命令来升级RKE：

    ```bash
    brew upgrade rke
    ```

## 二、准备Kubernetes集群的节点

Kubernetes集群组件在Linux系统上以docker容器的形式运行，您可以使用熟悉的Linux发行版，只要它可以满足Docker和Kubernetes的运行需要。

## 三、创建rke配置文件

有两种简单的方法可以创建`cluster.yml`：

- 使用我们的最小值rke配置[cluster.yml]({{< baseurl >}}/rke/latest/cn/example-yamls/#最小-cluster-yml-示例)并根据将使用的节点更新它；
- 使用`rke config`向导式生成配置；

### 1、运行`rke config`配置向导

```bash
./rke_darwin-amd64 config
```

### 2、指定名称创建配置文件

```bash
rke config --name cluster.yml
```

### 3、创建空的`cluster.yml`

如果需要空的`cluster.yml`模板，可以使用该`--empty`参数生成空模板。

```bash
rke config --empty --name cluster.yml
```

### 4、仅打印`cluster.yml`

您可以使用`--print`参数将生成的配置打印到stdout，而不是创建文件。

```bash
rke config --print
```

## 四、高可用性

RKE支持Kubernetes集群HA方式部署，您可以在`cluster.yml`文件中指定多个`controlplane`节点。RKE将在这些节点上部署`master`组件，并且kubelet配置为默认连接`127.0.0.1:6443`，这是`nginx-proxy`代理向所有主节点请求的服务的地址。

## 五、证书

_从v0.2.0版本起可用_

默认情况下，Kubernetes集群配置ssl证书来通信认证，RKE会自动为所有集群组件生成证书，部署Kubernetes集群后，您可以[管理这些自动生成的证书]({{< baseurl >}}/rke/latest/cn/cert-mgmt/#certificate-rotation)，您也可以使用[自定义证书]({{< baseurl >}}/rke/latest/cn/installation/custom-certs/)。

## 六、RKE部署Kubernetes集群

创建`cluster.yml`完成后，可以使用简单的命令部署集群。此命令假定该`cluster.yml`文件与运行该命令的目录位于同一目录中。

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

当最后一行显示`Finished building Kubernetes cluster successfully`表示集群已部署完成。作为Kubernetes创建过程的一部分，已创建并编写了一个`kubeconfig`文件，该文件`kube_config_cluster.yml`用于与Kubernetes集群进行交互。 如果您使用了不同的`cluster.yml`文件名，则kube配置文件将以`kube_config_<RKE_FILE_NAME>.yml`命名

## 七、保存文件

>**重要提示** 后期的故障排除和集群升级都需要以下文件

将以下文件的副本保存在安全位置：

- `cluster.yml`：RKE集群配置文件。

- `kube_config_cluster.yml`：集群的[Kubeconfig文件]({{< baseurl >}}/rke/latest/cn/kubeconfig/)，此文件包含完全访问集群的凭据。

- `cluster.rkestate`：[Kubernetes集群状态文件]({{< baseurl >}}/rke/latest/cn/installation/#八-kubernetes集群状态文件)，此文件包含访问集群的重要凭据。

  *使用`RKE v0.2.0或更高版本`时才会创建Kubernetes集群状态文件。*

## 八、Kubernetes集群状态文件

Kubernetes集群状态由Kubernetes集群中的集群配置文件`cluster.yml`和`组件证书`组成，由RKE保存，但根据您的RKE版本，集群状态的保存方式不同。

- 在v0.2.0之前，RKE将Kubernetes集群状态保存为`secret`。更新状态时，RKE会提取`secret`，`更新/更改`状态并保存新`secret`。
- 从v0.2.0开始，RKE在集群配置文件`cluster.yml`的同一目录中创建一个`.rkestate`文件。该`.rkestate`文件包含群集的当前状态，包括`RKE配置和证书`。需要保留此文件以更新群集或通过RKE对集群执行任何操作。
