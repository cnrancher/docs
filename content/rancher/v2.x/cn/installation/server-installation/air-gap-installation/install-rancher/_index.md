---
title: 2 — 安装Rancher
weight: 2
---

{{% tabs %}}
{{% tab "HA安装" %}}
## 一、RKE构建K8S集群

| 参数 | 说明                                                           |
| ----------------------- | --------------------------------------------------------------------- |
| `address`               | 节点公共IP地址|
| `internal_address`      | 节点内部IP地址     |
| `url`                   | 私有镜像仓库地址                                   |

```yaml
nodes:
- address: 18.222.121.187           # air gap node external IP
  internal_address: 172.31.7.22   # air gap node internal IP
  user: rancher
  role: [ "controlplane", "etcd", "worker" ]
  ssh_key_file: /home/user/.ssh/id_rsa
- address: 18.220.193.254           # air gap node external IP
  internal_address: 172.31.13.132 # air gap node internal IP
  user: rancher
  role: [ "controlplane", "etcd", "worker" ]
  ssh_key_file: /home/user/.ssh/id_rsa
- address: 13.59.83.89              # air gap node external IP
  internal_address: 172.31.3.216  # air gap node internal IP
  user: rancher
  role: [ "controlplane", "etcd", "worker" ]
  ssh_key_file: /home/user/.ssh/id_rsa
# 如果镜像仓库是私有仓库，需要配置仓库的登录凭证信息
private_registries:
- url: <REGISTRY.YOURDOMAIN.COM:PORT> # private registry url
  user: rancher
  password: "*********"
  is_default: true
```

### 运行RKE命令

```plain
rke up --config ./rancher-cluster.yml
```

### 测试K8S集群

按照[检查群集Pod的运行状况l]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install/helm-rancher/rke-install-k8s/#四-检查群集pod的运行状况)检查集群是健康状态。

## 二、Helm安装

>**注意** helm使用需要kubectl，点击了解[安装和配置kubectl]({{< baseurl >}}/rancher/v2.x/cn/installation/kubectl/)。

1. 配置tiller访问权限

    Helm在集群上安装tiller服务以管理charts. 由于RKE默认启用RBAC, 因此我们需要使用kubectl来创建一个serviceaccount，clusterrolebinding才能让tiller具有部署到集群的权限。

    ```bash
    kubectl -n kube-system create serviceaccount tiller
    kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
    ```
2. 创建registry secret(可选）

    Helm初始化的时候会去拉取`tiller`镜像，如果镜像仓库为私有仓库，则需要配置登录凭证。

    ```bash
    kubectl -n kube-system create secret docker-registry regcred \
    --docker-server="reg.example.com" \
    --docker-username=<user> \
    --docker-password=<password> \
    --docker-email=<email>
    ```
3. Patch the ServiceAccount

    ```bash
    kubectl -n kube-system patch serviceaccount tiller -p '{"imagePullSecrets": [{"name\": "regcred"}]}'
    ```
4. 安装Helm客户端

    参考[安装Helm客户端]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install/helm-rancher/helm-install/#二-安装helm客户端)了解Helm客户端安装。

5. 安装Tiller Server

    >修改镜像地址

    ```bash
    helm init --service-account tiller   --tiller-image reg.example.com/xxx/tiller:v2.11.0
    ```

## 三、安装Rancher

如果您在RKE中设置了默认的私有仓库凭证信息，那么Kubernetes `kubelet` 将使用这个凭证去登录仓库获取镜像。

### Charts 模板

需要在能连接互联网的主机上执行以下操作。

#### 1、Cert-Manager

如果要使用Rancher自签名证书安装Rancher，则需要在群集上安装`cert-manager`。如果您要安装自己的证书，可以跳过本节。

1. 获取`cert-manager`Charts离线包,访问[官方Helm chart仓库](https://github.com/helm/charts/tree/master/stable)查看查看信息。

    ```plain
    helm fetch stable/cert-manager
    ```
    将会在当前目录看到`cert-manager-vx.x.x.tgz`文件

2. 刷新`cert-manager`模板信息。设置`image.repository`以替换为离线镜像。

    ```plain
    helm template ./cert-manager-<version>.tgz --output-dir . \
    --name cert-manager --namespace kube-system \
    --set image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/jetstack/cert-manager-controller
    ```
    >**注意** 会在本地生成`cert-manager`文件夹 ，[cert-manager版本查询](https://github.com/helm/charts/blob/master/stable/cert-manager/values.yaml#L6)。

#### 2、Rancher

1. 添加Rancher Charts仓库。根据需要安装的版本(i.e. `latest` or `stable`)替换`<CHART_REPO>`，可通过[版本选择]({{< baseurl >}}/ancher/v2.x/cn/installation/server-tags/)查看版本说明。

    ```plain
    helm repo add rancher-<CHART_REPO> https://releases.rancher.com/server-charts/<CHART_REPO>
    ```

2. 获取Rancher Charts离线包。根据需要安装的版本(i.e. `latest` or `stable`)替换`<CHART_REPO>`，可通过[版本选择]({{< baseurl >}}/ancher/v2.x/cn/installation/server-tags/)查看版本说明。

    ```plain
    helm fetch rancher-<CHART_REPO>/rancher
    ```
3. 刷新模板信息。

    参考[在线Helm安装Rancher]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install/helm-rancher/rancher-install/)的方法，配置好相应的参数。

    ```plain
    helm template ./rancher-<version>.tgz --output-dir . \
    --name rancher --namespace cattle-system \
    --set hostname=<RANCHER.YOURDOMAIN.COM> \
    --set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:stable
    ```
    >**注意** 会在本地生成`rancher`文件夹

### 复制并应用配置文件

复制生成的`rancher`、`cert-manager`文件夹到可以访问本地K8S环境的主机上，通过`kubectl`应用配置。

```plain
kubectl create namespace cattle-system
kubectl -n kube-system apply -R -f ./cert-manager
kubectl -n cattle-system apply -R -f ./rancher
```

{{% /tab %}}
{{% tab "独立容器安装" %}}

根据[独立容器安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/single-node-install/)中的说明完成Rancher的安装；

> **注意**
>需要在镜像名之前添加镜像仓库地址

```plain
docker run -d --restart=unless-stopped \
 -p 80:80 -p 443:443 \
 <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
```

{{% /tab %}}
{{% /tabs %}}

## [下一步: 配置私有仓库凭证](../config-rancher-for-private-reg/)