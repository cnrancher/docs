---
title: 4 - 安装Rancher
weight: 4
aliases:
  - /rancher/v2.x/en/installation/air-gap-installation/install-rancher/
---

## 一、初始化Helm

>**注意** Helm运行需要依赖`kubectl`，点击了解[安装和配置kubectl]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/kubectl/)。

1. 配置Helm客户端访问权限

    Helm在Kubernetes集群上安装`Tiller`服务以管理charts,由于RKE默认启用RBAC, 因此我们需要使用kubectl来创建一个`serviceaccount`，`clusterrolebinding`才能让Tiller具有部署到集群的权限。

    `在离线环境中安装有kubectl的主机上执行以下命令`：

    ```bash
    # 指定kubeconfig配置文件
    kubeconfig=kube_configxxx.yml

    kubectl --kubeconfig=$kubeconfig -n kube-system \
        create serviceaccount tiller
    kubectl --kubeconfig=$kubeconfig \
        create clusterrolebinding tiller \
        --clusterrole cluster-admin \
        --serviceaccount=kube-system:tiller
    ```

1. 创建registry secret(可选）

    - Helm初始化的时候会去拉取`tiller`镜像，如果镜像仓库为私有仓库，则需要配置登录凭证。在离线环境中安装有kubectl的主机上执行以下命令：

        ```bash
        # 指定kubeconfig配置文件
        kubeconfig=kube_configxxx.yml

        kubectl --kubeconfig=$kubeconfig -n kube-system \
            create secret docker-registry regcred \
            --docker-server="reg.example.com" \
            --docker-username=<user> \
            --docker-password=<password> \
            --docker-email=<email>
        ```

    - Patch the ServiceAccount

        ```bash
        # 指定kubeconfig配置文件
        kubeconfig=kube_configxxx.yml

        kubectl --kubeconfig=$kubeconfig -n kube-system \
            patch serviceaccount tiller -p '{"imagePullSecrets": [{"name\": "regcred"}]}'
        ```

1. 安装Helm客户端

    在离线环境中安装有kubectl的主机上安装Helm客户端，参考[安装Helm客户端]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/helm-install/#二-安装helm客户端)了解Helm客户端安装。

1. 安装Helm Server(Tiller)

    >**注意:**
1、`helm init`在缺省配置下，会去谷歌镜像仓库拉取`gcr.io/kubernetes-helm/tiller`镜像，并在Kubernetes集群上安装配置Tiller。离线环境下，可通过`--tiller-image`指定私有镜像仓库镜像。\
2、`helm init`在缺省配置下，会利用`https://kubernetes-charts.storage.googleapis.com`作为缺省的`stable repository`地址,并去更新相关索引文件。\
3、如果您是离线安装`Tiller`, 如果有内部的`chart`仓库，可通过`--stable-repo-url`指定内部`chart`地址；如果没有内部的`chart`仓库, 可通过添加`--skip-refresh`参数禁止`Tiller`更新索引。

    `在离线环境中安装有kubectl和Helm客户端的主机上执行以下命令`

    ```bash
    kubeconfig=xxx.yml

    helm_version=`helm version |grep Client | awk -F""\" '{print $2}'`
    helm init --kubeconfig=$kubeconfig --skip-refresh \
        --service-account tiller \
        --tiller-image registry.cn-shanghai.aliyuncs.com/rancher/tiller:${helm_version}
    ```

## 二、打包Rancher Charts模板

此步骤需要在能连接互联网的主机上操作，在可访问互联网并安装有Helm客户端的主机上执行以下操作。

1. 添加`Rancher Charts`仓库。

    指定安装的版本(比如: `latest` or `stable`)，可通过[版本选择]({{< baseurl >}}/rancher/v2.x/cn/installation/server-tags/)查看版本说明。

    ```plain
    helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
    ```

2. 获取`Rancher Charts`离线包。

    指定安装的版本(比如: `latest`或`stable`或者通过`--version`指定版本)，可通过[版本选择]({{< baseurl >}}/rancher/v2.x/cn/installation/server-tags/)查看版本说明。

    ```plain
    helm fetch rancher-stable/rancher --version v2.2.3
    ```

    >**结果** 默认在当前目录下生成`rancher-vx.x.x.tgz`压缩文件，可通过`-d`指定生成的压缩包路径，比如:`helm fetch rancher-stable/rancher --version v2.2.3 -d /home`，将会在`/home`目录下生成`rancher-vx.x.x.tgz`压缩文件。

## 三、离线安装Rancher

将生成的`rancher-vx.x.x.tgz`文件拷贝到离线环境安装有`kubectl`和Helm客户端的主机上，解压`rancher-vx.x.x.tgz`文件获得`rancher`文件夹。

- 以TCP L4负载均衡器或者ingress作为访问入口

1. 使用`权威认证证书`

    将`服务证书和CA中间证书链`合并名为`tls.crt`的文件中,将`私钥`复制到名为`tls.key`的文件中。使用kubectl创建类型为`tls`的`secrets`。

    ```bash
    # 指定kubeconfig配置文件
    kubeconfig=kube_configxxx.yml
    kubectl --kubeconfig=$kubeconfig \
        create namespace cattle-system

    kubectl --kubeconfig=$kubeconfig \
        -n cattle-system create secret \
        tls tls-rancher-ingress \
        --cert=./tls.crt \
        --key=./tls.key
    ```

    > **注意** 必须把服务证书文件和key文件重命名为`tls.crt`和`tls.key`。

    然后执行以下命令进行rancher安装:

    ```bash
    # 指定kubeconfig配置文件

    kubeconfig=kube_configxxx.yml

    helm --kubeconfig=$kubeconfig install ./rancher \
        --name rancher \
        --namespace cattle-system \
        --set hostname=<修改为自己的域名> \
        --set ingress.tls.source=secret
        --set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:stable
    ```

1. 使用`自己的自签名证书`

    如果使用的是自己创建的自签名证书，则需要创建`证书secret和CA证书secret`。

    ```plain
    # 指定kubeconfig配置文件

    kubeconfig=kube_configxxx.yml
    kubectl --kubeconfig=$kubeconfig \
        create namespace cattle-system

    kubectl --kubeconfig=$kubeconfig \
        -n cattle-system create \
        secret tls tls-rancher-ingress \
        --cert=./tls.crt \
        --key=./tls.key

    kubectl --kubeconfig=$kubeconfig \
        -n cattle-system \
        create secret generic tls-ca \
        --from-file=cacerts.pem
    ```

    > **注意** 必须保证文件名为`cacerts.pem`、`tls.crt`和`tls.key`。

    然后执行以下命令进行rancher安装

    ```plain
    # 指定kubeconfig配置文件

    kubeconfig=kube_configxxx.yml

    helm --kubeconfig=$kubeconfig install ./rancher \
        --name rancher \
        --namespace cattle-system \
        --set hostname=<修改为自己的域名> \
        --set ingress.tls.source=secret \
        --set privateCA=true \
        --set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:stable
    ```

- 以外部HTTP L7负载均衡器作为访问入口（nginx为例）

1. 使用`权威认证证书`

    使用外部七层负载均衡器作为访问入口，那么将需要把ssl证书配置在L7负载均衡器上面，如果是`权威认证证书`，rancher侧则无需配置证书。参考[nginx配置示例]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/https-l7/#nginx配置示例)了解L7负载均衡器证书配置方法。

    ```plain
    # 指定kubeconfig配置文件
    kubeconfig=kube_configxxx.yml

    helm --kubeconfig=$kubeconfig install ./rancher \
        --name rancher \
        --namespace cattle-system \
        --set hostname=<修改为自己的域名> \
        --set tls=external \
        --set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:stable
    ```

1. 使用`自己的自签名证书`

    使用外部七层负载均衡器作为访问入口，那么将需要把ssl证书配置在L7负载均衡器上面，如果是`自己的自签名证书`，则需要把`CA证书`以密文的形式导入rancher。参考[nginx配置示例]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/https-l7/#nginx配置示例)了解L7负载均衡器证书配置方法。

    ```plain
    # 指定kubeconfig配置文件

    kubeconfig=kube_configxxx.yml
    kubectl --kubeconfig=$kubeconfig \
        create namespace cattle-system

    kubectl --kubeconfig=$kubeconfig \
        -n cattle-system create \
        secret generic tls-ca \
        --from-file=cacerts.pem
    ```

    > **注意** 必须保证文件名为`cacerts.pem`。

    然后执行以下命令进行rancher安装

    ```plain
    # 指定kubeconfig配置文件
    kubeconfig=kube_configxxx.yml

    helm --kubeconfig=$kubeconfig install ./rancher \
        --name rancher \
        --namespace cattle-system \
        --set hostname=<修改为自己的域名> \
        --set privateCA=true \
        --set tls=external \
        --set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:stable
    ```

## 四、为Cluster Pod添加主机别名(/etc/hosts)(可选)

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

    ```plain
    #指定kubectl配置文件
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

    ```plain
    #指定kubectl配置文件
    export kubeconfig=xxx/xxx/xx.kubeconfig.yml

    kubectl --kubeconfig=$kubeconfig -n cattle-system \
    patch daemonsets cattle-node-agent --patch '{
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

    > 1、替换其中的域名和IP \
      2、注意json中的引号。