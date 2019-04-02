---
title: 4 - Helm安装Rancher
weight: 4
---

>**注意：** 对于没有Internet访问的系统，请参考[离线安装Rancher]({{< baseurl >}}/rancher/v2.x/cn/installation/air-gap-installation/)。

## 一、添加Chart仓库地址

使用`helm repo add`命令添加Rancher chart仓库地址,访问[Rancher tag和Chart版本]({{< baseurl >}}/rancher/v2.x/cn/installation/server-tags/)

替换`<CHART_REPO>`为您要使用的Helm仓库分支(即latest或stable）。

```bash
helm repo add rancher-stable \
https://releases.rancher.com/server-charts/stable
```

## 二、安装证书管理器(可选)

>**注意:** 只有Rancher自动生成的证书和LetsEncrypt颁发的证书才需要`cert-manager`。如果是你自己的证书，可使用`ingress.tls.source=secret`参数指定证书，并跳过此步骤。
> **重要提示:** 由于Helm v2.12.0和cert-manager存在问题，请使用Helm v2.12.1或更高版本。

Rancher依靠`Kubernetes Helm stable`仓库中的[cert-manager](https://github.com/kubernetes/charts/tree/master/stable/cert-manager)来颁发自签名或LetsEncrypt证书.

从Helm stable目录安装`cert-manager`。

```bash
helm install stable/cert-manager \
  --name cert-manager \
  --namespace kube-system
  --version v0.5.2
```

## 三、选择SSL配置方式并安装Rancher server

Rancher server设计默认需要开启SSL/TLS配置来保证安全。

证书来源有三种选择：

- `rancher` - 使用Rancher自动生成`CA/Certificates`证书(默认)。
- `letsEncrypt` - 使用`LetsEncrypt`颁发证书。
- `secret` - 使用`Kubernetes Secret`证书配置文件（推荐）。

### 1、Rancher自动生成证书(默认)

默认情况下，Rancher会自动生成`CA根证书`并使用`cert-manager`颁发证书以访问Rancher server界面。

唯一的要求是将`hostname`配置为访问Rancher的域名地址，使用这种SSL证书配置方式需提前安装[证书管理器](#二-安装证书管理器-可选)。

>修改`hostname`

```bash
helm install rancher-stable/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=<你自己的域名> \
```

### 2、LetsEncrypt

使用[LetsEncrypt](https://letsencrypt.org/)的免费服务发布可信的SSL证书。此配置需要使用http验证，因此给Rancher配置的访问域名地址必须是能够被`互联网访问`。使用这种SSL证书配置方式需提前安装[证书管理器](#二-安装证书管理器-可选)。

>修改`hostname`、`LetsEncrypt`

```bash
helm install rancher-stable/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=<你自己的域名> \ \
  --set ingress.tls.source=letsEncrypt \
  --set letsEncrypt.email=me@example.org
```

### 3、Kubernetes Secret证书

首先创建证书密文，以便`Rancher和Ingress Controller`可以使用。

- 权威CA机构颁发的证书

1. 将`服务证书`和`CA中间证书链`合并到一个名为`tls.crt`,将`私钥`复制到或者重命名为`tls.key`。

1. 使用`kubectl`与`tls`类型来创建`secrets`.

    >**注意：** 证书名称和密文名称不能改变

    ```bash
    kubectl --kubeconfig=kube_configxxx.yml create  namespace cattle-system
    kubectl --kubeconfig=kube_configxxx.yml \
    -n cattle-system \
    create secret tls tls-rancher-ingress \
    --cert=./tls.crt \
    --key=./tls.key
    ```

- 自签名ssl证书(可选)

1. 如果使用的是自签名ssl证书，则需要把CA文件给Rancher。

1. 将CA证书复制到或者重命名为`cacerts.pem`，并用`kubectl`在命名空间`cattle-system`中创建`tls-ca`secret。

    >**注意：** 证书名称和密文名称不能改变

    ```bash
    kubectl --kubeconfig=kube_configxxx.yml create  namespace cattle-system
    kubectl --kubeconfig=kube_configxxx.yml \
    -n cattle-system create secret generic tls-ca \
    --from-file=cacerts.pem
    ```

    >修改`hostname`

    ```bash
    helm install rancher-stable/rancher \
      --name rancher \
      --namespace cattle-system \
      --set hostname=<你自己的域名> \ \
      --set ingress.tls.source=secret
    ```

>**注意:**
1.证书对应的`域名`需要与`hostname`选项匹配，否则`ingress`将无法代理访问Rancher。\
2.如果你使用的是私有CA签名证书，需要添加`--set privateCA=true`

### 4、高级配置

Rancher chart有许多配置选项,可用于自定义安装以适合你的特定环境，点击查看[Rancher高级设置](../advanced-settings)

### 5、保存配置参数

确保保存了所有的配置参数，Rancher下一次升级时，helm需要使用相同的配置参数来运行新版本Rancher。

## 6、(可选)为Agent Pod添加主机别名(/etc/hosts)

如果你没有内部DNS服务器而是通过添加`/etc/hosts`主机别名的方式指定的Rancher server域名，那么不管通过哪种方式(自定义、导入、Host驱动等)创建K8S集群，K8S集群运行起来之后，因为`cattle-cluster-agent Pod`和`cattle-node-agent`无法通过DNS记录找到`Rancher server`,最终导致无法通信。

### 解决方法

可以通过给`cattle-cluster-agent Pod`和`cattle-node-agent`添加主机别名(/etc/hosts)，让其可以正常通信`(前提是IP地址可以互通)`。

1. cattle-cluster-agent pod

    ```bash
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml #指定kubectl配置文件
    kubectl --kubeconfig=kube_configxxx.yml -n   cattle-system patch  deployments cattle-cluster-agent --patch '{
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

    ```bash
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml #指定kubectl配置文件
    kubectl --kubeconfig=kube_configxxx.yml -n   cattle-system patch  daemonsets cattle-node-agent --patch '{
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

四、故障排除

[故障排除]({{< baseurl >}}/rancher/v2.x/cn/faq/troubleshooting-helm/)
