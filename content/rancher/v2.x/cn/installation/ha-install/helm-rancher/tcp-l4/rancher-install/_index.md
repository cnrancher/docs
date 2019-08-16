---
title: 4 - Helm安装Rancher
weight: 4
---

>**注意：** 对于离线环境，请参考[离线安装Rancher]({{< baseurl >}}/rancher/v2.x/cn/installation/air-gap-installation/)。

## 一、添加Chart仓库地址

使用`helm repo add`命令添加Rancher chart仓库地址,访问[Rancher tag和Chart版本]({{< baseurl >}}/rancher/v2.x/cn/installation/server-tags/)

替换`<CHART_REPO>`为您要使用的Helm仓库分支(即latest或stable）。

```bash
helm repo add rancher-stable \
https://releases.rancher.com/server-charts/stable
```

## 二、使用自签名或者权威SSL证书安装Rancher Server

Rancher Server设计默认需要开启SSL/TLS配置来保证安全，将ssl证书以`Kubernetes Secret`卷的形式传递给`Rancher Server或Ingress Controller`。首先创建证书密文，以便`Rancher和Ingress Controller`可以使用。

{{% accordion id="option-a1" label="1、使用权威CA机构颁发的证书" %}}

1. 将`服务证书`和`CA中间证书链`合并到`tls.crt`,将`私钥`复制到或者重命名为`tls.key`；

1. 使用`kubectl`创建`tls`类型的`secrets`；

    >**注意:** `证书、私钥`名称必须是`tls.crt、tls.key`。

    ```bash
    # 指定配置文件
    export kubeconfig=xxx/xxx/xx.kubeconfig.yml

    kubectl --kubeconfig=$kubeconfig \
        create namespace cattle-system
    kubectl --kubeconfig=$kubeconfig \
        -n cattle-system create \
        secret tls tls-rancher-ingress \
        --cert=./tls.crt \
        --key=./tls.key
    ```

1. 安装Rancher Server

    >修改`hostname`

    ```bash
    # 指定配置文件
    export kubeconfig=xxx/xxx/xx.kubeconfig.yml
    helm --kubeconfig=$kubeconfig install \
        rancher-stable/rancher \
        --name rancher \
        --namespace cattle-system \
        --set hostname=<您自己的域名> \
        --set ingress.tls.source=secret
    ```

    >**注意:** 1.创建证书对应的`域名`需要与`hostname`选项匹配，否则`ingress`将无法代理访问Rancher。

{{% /accordion %}}
{{% accordion id="option-a2" label="2、使用自签名ssl证书(可选)" %}}

1. 如果没有自签名ssl证书，可以参考[自签名ssl证书]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/self-signed-ssl/#四-生成自签名证书)，一键生成ssl证书；

1. 一键生成ssl自签名证书脚本将自动生成`tls.crt、tls.key、cacerts.pem`三个文件，文件名称不能修改。如果使用您自己生成的自签名ssl证书，则需要将`服务证书`和`CA中间证书链`合并到`tls.crt`文件中,将`私钥`复制到或者重命名为`tls.key`文件，将`CA证书`复制到或者重命名为`cacerts.pem`。

1. 使用`kubectl`在命名空间`cattle-system`中创建`tls-ca`和`tls-rancher-ingress`两个`secret`;

    >**注意:** `证书、私钥、ca`名称必须是`tls.crt、tls.key、cacerts.pem`。

    ```bash
    # 指定配置文件
    kubeconfig=xxx/xxx/xx.kubeconfig.yml
    # 创建命名空间
    kubectl --kubeconfig=$kubeconfig \
        create namespace cattle-system
    # 服务证书和私钥密文
    kubectl --kubeconfig=$kubeconfig \
        -n cattle-system create \
        secret tls tls-rancher-ingress \
        --cert=./tls.crt \
        --key=./tls.key
    # ca证书密文
    kubectl --kubeconfig=$kubeconfig \
        -n cattle-system create secret \
        generic tls-ca \
        --from-file=cacerts.pem
    ```

1. 安装Rancher Server

    >修改`hostname`

    ```bash
    kubeconfig=xxx/xxx/xx.kubeconfig.yml

    helm --kubeconfig=$kubeconfig install \
        rancher-stable/rancher \
        --name rancher \
        --namespace cattle-system \
        --set hostname=<您自己的域名> \
        --set ingress.tls.source=secret \
        --set privateCA=true
    ```

    >**注意:** 1.证书对应的`域名`需要与`hostname`选项匹配，否则`ingress`将无法代理访问Rancher。

{{% /accordion %}}

### 4、高级配置

Rancher chart有许多配置选项,可用于自定义安装以适合您的特定环境，点击查看[Rancher高级设置](../advanced-settings)

### 5、保存配置参数

确保保存了所有的配置参数，Rancher下一次升级时，helm需要使用相同的配置参数来运行新版本Rancher。

### 6、(可选)为Agent Pod添加主机别名(/etc/hosts)

如果您没有内部DNS服务器而是通过添加`/etc/hosts`主机别名的方式指定的Rancher Server域名，那么不管通过哪种方式(自定义、导入、Host驱动等)创建K8S集群，K8S集群运行起来之后，因为`cattle-cluster-agent Pod`和`cattle-node-agent`无法通过DNS记录找到`Rancher Server URL`,最终导致无法通信。

### 解决方法

可以通过给`cattle-cluster-agent Pod`和`cattle-node-agent`添加主机别名(/etc/hosts)，让其可以正常通过`Rancher Server URL`与Rancher Server通信`(前提是IP地址可以互通)`。

- 操作步骤

1. `cattle-cluster-agent Pod`和`cattle-node-agent`需要在`LOCAL`集群初始化之后才会部署，所以先通过`Rancher Server URL`访问Rancher Web UI进行初始化。
1. 执行以下命令为Rancher Server容器配置hosts:

    ```bash
    #指定kubectl配置文件
    export kubeconfig=xxx/xxx/xx.kubeconfig.yml

    kubectl --kubeconfig=$kubeconfig -n cattle-system \
        patch deployments rancher --patch '{
            "spec": {
                "template": {
                    "spec": {
                        "hostAliases": [
                            {
                                "hostnames":
                                [
                                    "xxx.cnrancher.com"
                                ],
                                    "ip": "192.168.1.100"
                            }
                        ]
                    }
                }
            }
        }'
    ```

1. 通过`Rancher Server URL`访问Rancher Web UI，设置用户名密码和`Rancher Server URL`地址，然后会自动登录Rancher Web UI；
1. 在Rancher Web UI中依次进入`local集群/system项目`，在`cattle-system`命名空间中查看是否有`cattle-cluster-agent Pod`和`cattle-node-agent`被创建。如果有创建则进行下面的步骤，没有创建则等待；

1. cattle-cluster-agent pod

    ```bash
    export kubeconfig=xxx/xxx/xx.kubeconfig.yml

    kubectl --kubeconfig=$kubeconfig -n cattle-system \
    patch deployments cattle-cluster-agent --patch '{
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

1. cattle-node-agent pod

    ```bash
    export kubeconfig=xxx/xxx/xx.kubeconfig.yml

    kubectl --kubeconfig=$kubeconfig -n cattle-system \
    patch  daemonsets cattle-node-agent --patch '{
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

## 三、故障排除

[故障排除]({{< baseurl >}}/rancher/v2.x/cn/faq/troubleshooting-helm/)
