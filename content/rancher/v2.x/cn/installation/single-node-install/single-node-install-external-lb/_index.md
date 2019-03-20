---
title: 1 - 独立容器安装+外部七层负载平衡
weight: 1
---

对于开发环境，我们推荐直接在主机上通过`docker run`的形式运行Rancher server容器。如果主机可以通过公网IP直接访问，可以参考[单节点安装]({{< baseurl >}}/rancher/v2.x/cn/installation/single-node-install)。

## 一、Linux主机要求

- [基础环境配置]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/basic-environment-configuration/)
- [端口需求]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/references/)

## 二、安装Rancher并配置SSL证书

出于安全考虑，使用Rancher时需要SSL进行加密。SSL可以保护所有Rancher网络通信，例如登录或与集群交互。

> **注意:**
> 1、需要[离线安装？]({{< baseurl >}}/rancher/v2.x/cn/installation/air-gap-installation/) \
> 2、需要开启[API审计日志？]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/api-auditing/) \
> 3、需要[代理上网?]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/proxy-configuration/)

{{% accordion id="1" label="方案A-使用你自己生成的自签名证书" %}}

如果你选择使用自签名证书来加密通信，则必须将证书安装在负载均衡器上，并且将CA证书放置于Rancher容器中。

> **先决条件:**
>1、创建一个自签名证书。\
>2、证书文件必须是PEM格式。\
>3、这里的证书不需要进行`base64`加密。

运行Rancher server容器时候，需要把自签名CA证书映射到Rancher容器中:

```bash
docker run -d --restart=unless-stopped \
-p 80:80 -p 443:443 \
-v /root/var/log/auditlog:/var/log/auditlog \
-e AUDIT_LEVEL=3 \
-v /etc/your_certificate_directory/cacerts.pem:/etc/rancher/ssl/cacerts.pem \
rancher/rancher:latest
```

{{% /accordion %}}
{{% accordion id="2" label="方案B-使用权威CA机构颁发的证书" %}}

如果你公开发布你的应用，理想情况下应该使用由权威CA机构颁发的证书。

> **先决条件:**
>1、证书必须是`PEM格式`,`PEM`只是一种证书类型，并不是说文件必须是PEM为后缀，具体可以查看[证书类型]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/self-signed-ssl/)；\
>2、这里的证书不需要进行`base64`加密;\
>3、给容器添加`--no-cacerts`参数禁止Rancher生成默认CA证书；

如果你使用由权威CA机构颁发的证书，则无需在Rancher容器中安装你的CA证书，只需运行下面的基本安装命令即可:

```bash
docker run -d --restart=unless-stopped \
-p 80:80 -p 443:443 \
-v /root/var/log/auditlog:/var/log/auditlog \
-e AUDIT_LEVEL=3 \
rancher/rancher:latest --no-cacerts
```

{{% /accordion %}}

## 三、配置七层负载均衡器

默认情况下，通过`docker run`运行的Rancher server容器会自动把端口80重定向到443，但是通过负载均衡器来代理Rancher server容器后，不再需要将Rancher server容器80端口重定向到443端口。通过在负载均衡器上配置`X-Forwarded-Proto: https`参数后，Rancher server容器端口重定向功能将自动被禁用。

负载均衡器或代理必须配置为支持以下内容:

- **WebSocket**连接
- **SPDY**/**HTTP/2**协议
- 传递/设置以下headers:

    | Header       | Value       | 描述     |
    |-----------|-------------|:-----------|
    | `Host`              | 传递给Rancher的主机名| 识别客户端请求的主机名。          |
    | `X-Forwarded-Proto` | `https`     | 识别客户端用于连接负载均衡器的协议。**注意：**如果存在此标头，`rancher/rancher`不会将HTTP重定向到HTTPS。 |
    | `X-Forwarded-Port`  | Port used to reach Rancher.  | 识别客户端用于连接负载均衡器的协议。   |
    | `X-Forwarded-For`   | IP of the client connection.  | 识别客户端的原始IP地址。  |

## 四、Nginx 配置文件示例

此Nginx配置文件在Nginx version 1.13 (mainline)和1.14(stable)通过测试

```bash
worker_processes 4;
worker_rlimit_nofile 40000;

events {
    worker_connections 8192;
}

http {
    upstream rancher {
        server rancher-server:80;
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
            # This allows the ability for the execute shell window to remain open for up to 15 minutes. Without this parameter, the default is 1 minute and will automatically close.
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

## 五、(可选)为Agent Pod添加主机别名(/etc/hosts)

如果你没有内部DNS服务器而是通过添加`/etc/hosts`主机别名的方式指定的Rancher server域名，那么不管通过哪种方式(自定义、导入、Host驱动等)创建K8S集群，K8S集群运行起来之后，因为`cattle-cluster-agent Pod`和`cattle-node-agent`无法通过DNS记录找到`Rancher server`,最终导致无法通信。

### 解决方法

可以通过给`cattle-cluster-agent Pod`和`cattle-node-agent`添加主机别名(/etc/hosts)，让其可以正常通信`(前提是IP地址可以互通)`。

1. cattle-cluster-agent pod

    ```bash
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml #指定kubectl配置文件
    kubectl -n cattle-system patch  deployments cattle-cluster-agent --patch '{
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

2. cattle-node-agent pod

    ```bash
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

## 六、FAQ和故障排除

[FAQ]({{< baseurl >}}/rancher/v2.x/cn/faq/)中整理了常见的问题与解决方法。
