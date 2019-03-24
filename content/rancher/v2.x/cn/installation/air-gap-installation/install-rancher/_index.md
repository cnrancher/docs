---
title: 2 — 离线安装Rancher
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

按照[检查集群Pod的运行状况l]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/rke-install-k8s/#四-检查集群pod的运行状况)检查集群是健康状态。

## 二、安装Helm Server和Helm 客户端

>**注意** Helm运行需要依赖`kubectl`，点击了解[安装和配置kubectl]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/kubectl/)。

1. 配置Helm客户端访问权限

    Helm在Kubernetes集群上安装`Tiller`服务以管理charts,由于RKE默认启用RBAC, 因此我们需要使用kubectl来创建一个`serviceaccount`，`clusterrolebinding`才能让Tiller具有部署到集群的权限。

    ```bash
    kubectl -n kube-system create serviceaccount tiller
    kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
    ```

1. 创建registry secret(可选）

    - Helm初始化的时候会去拉取`tiller`镜像，如果镜像仓库为私有仓库，则需要配置登录凭证。

        ```bash
        kubectl -n kube-system create secret docker-registry regcred \
        --docker-server="reg.example.com" \
        --docker-username=<user> \
        --docker-password=<password> \
        --docker-email=<email>
        ```

    - Patch the ServiceAccount

        ```bash
        kubectl -n kube-system patch serviceaccount tiller -p '{"imagePullSecrets": [{"name\": "regcred"}]}'
        ```

1. 安装Helm客户端

    参考[安装Helm客户端]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/helm-install/#二-安装helm客户端)了解Helm客户端安装。

1. 安装Helm Server(Tiller)

    >**注意:**
1、`helm init`在缺省配置下，会去谷歌镜像仓库拉取`gcr.io/kubernetes-helm/tiller`镜像，并在Kubernetes集群上安装配置Tiller。离线环境下，可通过`--tiller-image`指定私有镜像仓库镜像。点击查询[tiller镜像版本](https://hub.docker.com/r/hongxiaolu/tiller/tags/)。\
2、`helm init`在缺省配置下，会利用`https://kubernetes-charts.storage.googleapis.com`作为缺省的`stable repository`地址,并去更新相关索引文件。如果你是离线安装`Tiller`, 如果有内部的`chart`仓库，可通过`--stable-repo-url`指定内部`chart`地址；如果没有内部的`chart`仓库, 可通过添加`--skip-refresh`参数禁止`Tiller`更新索引。

    ```bash
    export tag=<修改版本号>
    helm init --skip-refresh --service-account tiller   --tiller-image reg.example.com/google_containers/tiller:${tag}
    ```

    >**注意** 修改镜像名和版本号

## 三、配置Rancher离线模板

如果您在RKE中设置了默认的私有仓库凭证信息，那么Kubernetes `kubelet` 将使用这个凭证去登录仓库获取镜像。此步骤需要在能连接互联网的主机上操作。

### 1、打包Cert-Manager Charts模板（可选）

如果要使用`Rancher自签名证书安装Rancher`，则需要在集群上安装`cert-manager`。

>如果您要使用自己的证书(`自签名或者权威证书`)，可以跳过本节。

1. 获取`cert-manager`Charts离线包。

    可访问官方[Helm chart仓库](https://github.com/helm/charts/tree/master/stable)查看信息。

    ```plain
    helm fetch stable/cert-manager
    ```

    >打包下载`cert-manager Charts`文件，将会在当前目录看到`cert-manager-vx.x.x.tgz`文件。

2. 更新`cert-manager`模板配置。

    配置`image.repository`等离线参数信息，通过执行`helm template`命令生成相应的`yaml`配置文件。

    ```plain
    helm template ./cert-manager-<version>.tgz --output-dir . \
    --name cert-manager --namespace kube-system \
    --set image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/jetstack/cert-manager-controller
    ```

    >**注意** 命令执行后，会在当前目录生成`cert-manager`文件夹 。替换镜像地址，点击[cert-manager镜像版本查询](https://github.com/helm/charts/blob/master/stable/cert-manager/values.yaml#L6)。

### 2、打包Rancher Charts模板

1. 添加`Rancher Charts`仓库。

    指定安装的版本(比如: `latest` or `stable`)，可通过[版本选择]({{< baseurl >}}/rancher/v2.x/cn/installation/server-tags//)查看版本说明。

    ```plain
    helm repo add rancher-<CHART_REPO> https://releases.rancher.com/server-charts/<CHART_REPO>
    ```

2. 获取`Rancher Charts`离线包。

    指定安装的版本(比如: `latest` or `stable`)，可通过[版本选择]({{< baseurl >}}/rancher/v2.x/cn/installation/server-tags/)查看版本说明。

    ```plain
    helm fetch rancher-<CHART_REPO>/rancher
    ```

    >打包下载`Rancher Charts`文件，将会在当前目录看到`rancher-vx.x.x.tgz`文件。

### 3、更新`Rancher`模板配置。

通过执行`helm template`命令，生成相应的`yaml`配置文件。

1. 使用`Rancher自签名证书`

      ```plain
      helm template ./rancher-<version>.tgz --output-dir . \
      --name rancher \
      --namespace cattle-system \
      --set hostname=<RANCHER.YOURDOMAIN.COM> \
      --set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:stable
      ```

1. 使用`权威证书或者自己的自签名证书`

    - 添加`权威服务证书`secret

    将服务证书和CA中间证书链合并到一个名为`tls.crt`的文件中,将私钥复制到名为`tls.key`的文件中。使用kubectl创建`tls`类型的`secrets`。

    ```plain
    kubectl -n cattle-system create secret tls tls-rancher-ingress \
    --cert=./tls.crt \
    --key=./tls.key
    ```

    - 添加`自签名CA证书`secret

    如果使用的是自己创建的自签名证书，`需要创建CA文件的secret`。在`添加权威服务证书secret`基础上，执行以下步骤添加`自签名CA证书`secret：

    ```plain
    kubectl -n cattle-system create secret generic tls-ca \
    --from-file=cacerts.pem
    ```

    - **更新模板配置**

    ```plain
     helm template ./rancher-<version>.tgz --output-dir . \
     --name rancher \
     --namespace cattle-system \
     --set hostname=<RANCHER.YOURDOMAIN.COM> \
     --set ingress.tls.source=secret \
     --set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:stable
     ```

    >**注意** 执行`helm template`命令，会在当前目录生成`rancher`文件夹。

1. 使用外部7层负载均衡器作为SSL终止

## 四、离线安装Rancher

复制生成的`rancher`、`cert-manager`文件夹到可以访问本地K8S环境的主机上，通过`kubectl apply`执行配置。

```plain
kubectl create namespace cattle-system
kubectl -n kube-system apply -R -f ./cert-manager
kubectl -n cattle-system apply -R -f ./rancher
```

## 五、为Agent Pod添加主机别名(/etc/hosts)(可选)

如果你没有内部DNS服务器而是通过添加`/etc/hosts`主机别名的方式指定的Rancher server域名，那么不管通过哪种方式(自定义、导入、Host驱动等)创建K8S集群，K8S集群运行起来之后，因为`cattle-cluster-agent Pod`和`cattle-node-agent`无法通过DNS记录找到`Rancher server`,最终导致无法通信。

### 解决方法

可以通过给`cattle-cluster-agent Pod`和`cattle-node-agent`添加主机别名(/etc/hosts)，让其可以正常通信`(前提是IP地址可以互通)`。

1. cattle-cluster-agent pod

    ```plain
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml #指定kubectl配置文件
    kubectl -n cattle-system patch  deployments cattle-cluster-agent --patch '{
        "spec": {
            "template": {
                "spec": {
                    "hostAliases": [
                        {
                            "hostnames":
                            [
                                "demo.cnrancher.com"
                            ],
                                "ip": "192.168.1.100"
                        }
                    ]
                }
            }
        }
    }'
    ```

2. cattle-node-agent pod

    ```plain
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml #指定kubectl配置文件
    kubectl -n cattle-system patch  daemonsets cattle-node-agent --patch '{
        "spec": {
            "template": {
                "spec": {
                    "hostAliases": [
                        {
                            "hostnames":
                            [
                                "xxx.rancher.com"
                            ],
                                "ip": "192.168.1.100"
                        }
                    ]
                }
            }
        }
    }'
    ```

    > 注意json中的引号。

{{% /tab %}}
{{% tab "独立容器安装" %}}

根据[独立容器安装]({{< baseurl >}}/rancher/v2.x/cn/installation/single-node-install/)中的说明完成Rancher的安装；

> **注意**
>需要在镜像名之前添加镜像仓库地址

```plain
docker run -d --restart=unless-stopped \
 -p 80:80 -p 443:443 \
 <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
```

## (可选)为Agent Pod添加主机别名(/etc/hosts)

如果你没有内部DNS服务器而是通过添加`/etc/hosts`主机别名的方式指定的Rancher server域名，那么不管通过哪种方式(自定义、导入、Host驱动等)创建K8S集群，K8S集群运行起来之后，因为`cattle-cluster-agent Pod`和`cattle-node-agent`无法通过DNS记录找到`Rancher server`,最终导致无法通信。

### 解决方法

可以通过给`cattle-cluster-agent Pod`和`cattle-node-agent`添加主机别名(/etc/hosts)，让其可以正常通信`(前提是IP地址可以互通)`。

1. cattle-cluster-agent pod

    ```plain
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml #指定kubectl配置文件
    kubectl -n cattle-system patch  deployments cattle-cluster-agent --patch '{
        "spec": {
            "template": {
                "spec": {
                    "hostAliases": [
                        {
                            "hostnames":
                            [
                                "demo.cnrancher.com"
                            ],
                                "ip": "192.168.1.100"
                        }
                    ]
                }
            }
        }
    }'
    ```

2. cattle-node-agent pod

    ```plain
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml #指定kubectl配置文件
    kubectl -n cattle-system patch  daemonsets cattle-node-agent --patch '{
        "spec": {
            "template": {
                "spec": {
                    "hostAliases": [
                        {
                            "hostnames":
                            [
                                "xxx.rancher.com"
                            ],
                                "ip": "192.168.1.100"
                        }
                    ]
                }
            }
        }
    }'
    ```

    > **注意**
    >1、替换其中的域名和IP \
    >2、别忘记json中的引号。

{{% /tab %}}
{{% /tabs %}}

## [下一步: 配置私有仓库凭证](../config-rancher-for-private-reg/)