---
title: 2 - 七层负载均衡Helm HA部署
weight: 2
---

## 一、架构说明

![Rancher HA]({{< baseurl >}}/img/rancher/ha/rancher2ha-l7.svg)

## 二、配置负载均衡器(以NGINX为例)

默认情况下，如果rancher server通过负载均衡器来代理，这个时候请求是通过负载均衡器发送给rancher server，而并非客户端直接访问rancher server。在非全局`https`的环境中，如果以外部负载均衡器作为ssl终止，这个时候通过负载均衡器的`https`请求将需要被反向代理到rancher server http(80)上。在负载均衡器上配置`X-Forwarded-Proto: https`参数，rancher server http(80)上收到负载均衡器的请求后，就不会再重定向到https(443)上。

负载均衡器或代理必须支持以下参数:

- **WebSocket** 连接
- **SPDY**/**HTTP/2**协议
- 传递/设置以下headers:

| Header              | Value             | 描述            |
|---------------------|--------------------------|:------------|
| `Host`              | 传递给Rancher的主机名| 识别客户端请求的主机名。      |
| `X-Forwarded-Proto` | `https`       | 识别客户端用于连接负载均衡器的协议。**注意：**如果存在此标头，`rancher/rancher`不会将HTTP重定向到HTTPS。 |
| `X-Forwarded-Port`  | Port used to reach Rancher.   | 识别客户端用于连接负载均衡器的协议。      |
| `X-Forwarded-For`   | IP of the client connection.   | 识别客户端的原始IP地址。            |

### nginx配置示例

```plain
worker_processes 4;
worker_rlimit_nofile 40000;

events {
    worker_connections 8192;
}

http {
    upstream rancher {
        server IP_NODE_1:80;
        server IP_NODE_2:80;
        server IP_NODE_3:80;
    }

    map $http_upgrade $connection_upgrade {
        default Upgrade;
        ''      close;
    }

    server {
        listen 443 ssl http2;
        server_name FQDN;
        ssl_certificate /certs/fullchain.pem;
        ssl_certificate_key /certs/privkey.pem;

        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Port $server_port;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://rancher;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
            # This allows the ability for the execute shell window to remain open for up to 15 minutes. 
            ## Without this parameter, the default is 1 minute and will automatically close.
            proxy_read_timeout 900s;
            proxy_buffering off;
        }
    }

    server {
        listen 80;
        server_name FQDN;
        return 301 https://$server_name$request_uri;
    }
}
```

>为了减少网络传输的数据量，可以在七层代理的`http`定义中添加`GZIP`功能。

```bash
# Gzip Settings
gzip on;
gzip_disable "msie6";
gzip_disable "MSIE [1-6]\.(?!.*SV1)";
gzip_vary on;
gzip_static on;
gzip_proxied any;
gzip_min_length 0;
gzip_comp_level 8;
gzip_buffers 16 8k;
gzip_http_version 1.1;
gzip_types
  text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml application/font-woff
  text/javascript application/javascript application/x-javascript
  text/x-json application/json application/x-web-app-manifest+json
  text/css text/plain text/x-component
  font/opentype application/x-font-ttf application/vnd.ms-fontobject font/woff2
  image/x-icon image/png image/jpeg;
```

## 二、RKE安装Kubernetes

参考[RKE安装Kubernetes]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/rke-install-k8s/)

## 三、安装配置Helm

参考[安装配置Helm]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/helm-install/)

## 四、Helm安装Rancher

### 1、添加Chart仓库地址

使用`helm repo add`命令添加Rancher chart仓库,访问了解[Rancher tag和Chart版本]({{< baseurl >}}/rancher/v2.x/cn/installation/server-tags/)。

```bash
helm repo add rancher-stable \
https://releases.rancher.com/server-charts/stable
```

### 2、配置SSL并安装Rancher server

Rancher server设计默认需要开启SSL/TLS配置来保证安全。

因为选择外部七层负载均衡器作为ssl终止，那么后端的访问连接不就需要走https，所以Rancher server只需要把`80`端口暴露出去。并且如果外部七层负载均衡器作为ssl终止，那么Rancher server就不需要绑定SSL证书。但如果使用的是自签名SSL证书，需要把CA证书传递给Rancher。

- 使用权威CA机构颁发的证书

1. 根据[配置负载均衡器](#二-配置负载均衡器-以nginx为例)配置`服务证书`和`私钥`；

1. 安装rancher server

    >修改`hostname`

    ```bash
    # 指定配置文件
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml

    helm --kubeconfig=$KUBECONFIG install rancher-stable/rancher \
      --name rancher --namespace cattle-system \
      --set hostname=<你自己的域名> \
      --set tls=external
    ```

    >**注意:** 1.创建证书对应的`域名`需要与`hostname`选项匹配，否则`ingress`将无法代理访问Rancher。

- 使用自签名ssl证书(可选)

1. 如果没有自签名ssl证书，可以参考[自签名ssl证书]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/self-signed-ssl/#四-生成自签名证书)，一键生成ssl证书；

1. 一键生成ssl自签名证书脚本将自动生成`tls.crt、tls.key、cacerts.pem`三个文件，文件名称不能修改。如果使用你自己生成的自签名ssl证书，则需要将`服务证书`和`CA中间证书链`合并到`tls.crt`文件中,将`私钥`复制到或者重命名为`tls.key`文件，将`CA证书`复制到或者重命名为`cacerts.pem`。

1. 根据[配置负载均衡器](#二-配置负载均衡器-以nginx为例)配置`服务证书`和`私钥`；

1. 使用`kubectl`在命名空间`cattle-system`中创建`tls-ca`secret。

    >**注意:** `ca`文件名称必须是`cacerts.pem`。

    ```bash
    # 指定配置文件
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml

    # 创建命名空间
    kubectl --kubeconfig=$KUBECONFIG create namespace cattle-system

    # ca证书密文
    kubectl --kubeconfig=$KUBECONFIG -n cattle-system create secret \
    generic tls-ca \
    --from-file=cacerts.pem
    ```

1. 安装rancher server

    >修改`hostname`

    ```bash
    # 指定配置文件
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml

    helm --kubeconfig=$KUBECONFIG install rancher-stable/rancher \
      --name rancher --namespace cattle-system \
      --set hostname=<你自己的域名> \
      --set tls=external \
      --set privateCA=true
    ```

    >**注意:** 1.证书对应的`域名`需要与`hostname`选项匹配，否则`ingress`将无法代理访问Rancher。

### 3、高级配

Rancher chart有许多配置选项,可用于自定义安装以适合你的特定环境，点击查看[Rancher高级设置]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/advanced-settings)

## 五、(可选)为Agent Pod添加主机别名(/etc/hosts)

如果你没有内部DNS服务器而是通过添加`/etc/hosts`主机别名的方式指定的Rancher server域名，那么不管通过哪种方式(自定义、导入、Host驱动等)创建K8S集群，K8S集群运行起来之后，因为`cattle-cluster-agent Pod`和`cattle-node-agent`无法通过DNS记录找到`Rancher server`,最终导致无法通信。

### 解决方法

可以通过给`cattle-cluster-agent Pod`和`cattle-node-agent`添加主机别名(/etc/hosts)，让其可以正常通信`(前提是IP地址可以互通)`。

1. cattle-cluster-agent pod

    ```bash
    #指定kubectl配置文件
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml

    kubectl --kubeconfig=$KUBECONFIG -n cattle-system \
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

2. cattle-node-agent pod

    ```bash
    #指定kubectl配置文件
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml

    kubectl --kubeconfig=$KUBECONFIG -n cattle-system \
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

## 六、FAQ和故障排除

[FAQ]({{< baseurl >}}/rancher/v2.x/cn/faq/)中整理了常见的问题与解决方法。
