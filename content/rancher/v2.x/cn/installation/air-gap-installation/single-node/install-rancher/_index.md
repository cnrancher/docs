---
title: 3 - 安装Rancher
weight: 3
---

出于安全考虑，使用Rancher时需要SSL进行加密。SSL可以保护所有Rancher网络通信，例如登录或与集群交互。

> 1. [自定义CA证书]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/custom-ca-root-certificate/)
> 2. [开启审计日志]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/api-auditing/)
> 3. [TLS设置]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/tls-setting/)

## 一、不通过代理直接访问Rancher server

1. 默认自签名证书安装

    默认情况下，Rancher会自动生成一个用于加密的自签名证书。从你的Linux主机运行Docker命令来安装Rancher，而不需要任何其他参数:

    ```bash
    docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    -v <主机路径>:/var/lib/rancher/ \
    -v /root/var/log/auditlog:/var/log/auditlog \
    -e AUDIT_LEVEL=3 \
    <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
    ```

1. 使用自己的自签名证书

    Rancher安装可以使用自己生成的自签名证书，如果没有自签名证书，可一键生成[自签名ssl证书]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/self-signed-ssl/)。

    > **先决条件:**
    > - 使用OpenSSL或其他方法创建自签名证书。\
    > - 这里的证书不需要进行`base64`加密。\
    > - 证书文件必须是[PEM]({{< baseurl >}}/rancher/v2.x/cn/installation/single-node-install/#我如何知道我的证书是否为pem格式)格式。\
    > - 在你的证书文件中，包含链中的所有中间证书。有关示例，请参考[SSL常见问题/故障排除]({{< baseurl >}}/rancher/v2.x/cn/installation/single-node-install/#如果我想添加我的中间证书-证书的顺序是什么)。

    你的Rancher安装可以使用你提供的自签名证书来加密通信。创建证书后，运行docker命令时把证书文件映射到容器中。

    ```bash
    docker run -d --restart=unless-stopped \
      -p 80:80 -p 443:443 \
      -v /var/log/rancher/auditlog:/var/log/auditlog \
      -e AUDIT_LEVEL=3 \
      -v <主机路径>:/var/lib/rancher/ \
      -v /etc/<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
      -v /etc/<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
      -v /etc/<CERT_DIRECTORY>/<CA_CERTS.pem>:/etc/rancher/ssl/cacerts.pem \
     <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
    ```

1. 使用权威CA机构颁发的证书

    如果你公开发布你的应用，理想情况下应该使用由权威CA机构颁发的证书。

    > **先决条件:**
    >1.证书必须是`PEM格式`,`PEM`只是一种证书类型，并不是说文件必须是PEM为后缀，具体可以查看[证书类型]({{< baseurl >}}/rancher/v2.x/cn/    install-prepare/self-signed-ssl/)。\
    >2.确保容器包含你的证书文件和密钥文件。由于你的证书是由认可的CA签署的，因此不需要安装额外的CA证书文件。\
    >3.给容器添加`--no-cacerts`参数禁止Rancher生成默认CA证书。\
    >4.这里的证书不需要进行`base64`加密。

    获取证书后，运行Docker命令以部署Rancher，同时指向证书文件。

    ```bash
    docker run -d --restart=unless-stopped \
      -p 80:80 -p 443:443 \
      -v <主机路径>:/var/lib/rancher/ \
      -v /root/var/log/auditlog:/var/log/auditlog \
      -e AUDIT_LEVEL=3 \
      -v /etc/your_certificate_directory/fullchain.pem:/etc/rancher/ssl/cert.pem \
      -v /etc/your_certificate_directory/privkey.pem:/etc/rancher/ssl/key.pem \
      <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG> \
      --no-cacerts
    ```

## 二、通过TCP L4层代理访问Rancher server

1. 首先在负载均衡器主机上安装NGINX，NGINX具有适用于所有已知操作系统的软件包。有关安装NGINX的帮助，请查阅其安装文档[nginx安装文档](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/).

1. 安装NGINX后，你需要使用节点的IP地址更新NGINX配置文件`nginx.conf`。复制下面的代码到文本编辑器，保存为`nginx.conf`。在`nginx.conf`配置中, 替换`IP_NODE_1`、`IP_NODE_2`、`IP_NODE_3` 为你主机真实的IP地址。

    **NGINX配置示例:**

    ```bash
    worker_processes 4;
    worker_rlimit_nofile 40000;

    events {
        worker_connections 8192;
    }

    stream {
        upstream rancher_servers_http {
            least_conn;
            server <IP_NODE_1>:80 max_fails=3 fail_timeout=5s;
            server <IP_NODE_2>:80 max_fails=3 fail_timeout=5s;
            server <IP_NODE_3>:80 max_fails=3 fail_timeout=5s;
        }
        server {
            listen     80;
            proxy_pass rancher_servers_http;
        }

        upstream rancher_servers_https {
            least_conn;
            server <IP_NODE_1>:443 max_fails=3 fail_timeout=5s;
            server <IP_NODE_2>:443 max_fails=3 fail_timeout=5s;
            server <IP_NODE_3>:443 max_fails=3 fail_timeout=5s;
        }
        server {
            listen     443;
            proxy_pass rancher_servers_https;
        }
    }
    ```

1. 保存 `nginx.conf` ，并复制`nginx.conf`到负载均衡器节点的`/etc/nginx/nginx.conf`路径下。
1. 重新加载nginx配置

    ```bash
    nginx -s reload
    ```

1. 默认自签名证书安装

    默认情况下，Rancher会自动生成一个用于加密的自签名证书。从你的Linux主机运行Docker命令来安装Rancher，而不需要任何其他参数:

    ```bash
    docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    -v <主机路径>:/var/lib/rancher/ \
    -v /root/var/log/auditlog:/var/log/auditlog \
    -e AUDIT_LEVEL=3 \
    <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
    ```

1. 使用自己的自签名证书

    Rancher安装可以使用自己生成的自签名证书，如果没有自签名证书，可一键生成[自签名ssl证书]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/self-signed-ssl/)。

    > **先决条件:**
    > - 使用OpenSSL或其他方法创建自签名证书。\
    > - 这里的证书不需要进行`base64`加密。\
    > - 证书文件必须是[PEM]({{< baseurl >}}/rancher/v2.x/cn/installation/single-node-install/#我如何知道我的证书是否为pem格式)格式。\
    > - 在你的证书文件中，包含链中的所有中间证书。有关示例，请参考[SSL常见问题/故障排除]({{< baseurl >}}/rancher/v2.x/cn/installation/single-node-install/#如果我想添加我的中间证书-证书的顺序是什么)。

    你的Rancher安装可以使用你提供的自签名证书来加密通信。创建证书后，运行docker命令时把证书文件映射到容器中。

    ```bash
    docker run -d --restart=unless-stopped \
      -p 80:80 -p 443:443 \
      -v /var/log/rancher/auditlog:/var/log/auditlog \
      -e AUDIT_LEVEL=3 \
      -v <主机路径>:/var/lib/rancher/ \
      -v /etc/<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
      -v /etc/<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
      -v /etc/<CERT_DIRECTORY>/<CA_CERTS.pem>:/etc/rancher/ssl/cacerts.pem \
     <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
    ```

1. 使用权威CA机构颁发的证书

    如果你公开发布你的应用，理想情况下应该使用由权威CA机构颁发的证书。

    > **先决条件:**
    >1.证书必须是`PEM格式`,`PEM`只是一种证书类型，并不是说文件必须是PEM为后缀，具体可以查看[证书类型]({{< baseurl >}}/rancher/v2.x/cn/    install-prepare/self-signed-ssl/)。\
    >2.确保容器包含你的证书文件和密钥文件。由于你的证书是由认可的CA签署的，因此不需要安装额外的CA证书文件。\
    >3.给容器添加`--no-cacerts`参数禁止Rancher生成默认CA证书。\
    >4.这里的证书不需要进行`base64`加密。

    获取证书后，运行Docker命令以部署Rancher，同时指向证书文件。

    ```bash
    docker run -d --restart=unless-stopped \
      -p 80:80 -p 443:443 \
      -v <主机路径>:/var/lib/rancher/ \
      -v /root/var/log/auditlog:/var/log/auditlog \
      -e AUDIT_LEVEL=3 \
      -v /etc/your_certificate_directory/fullchain.pem:/etc/rancher/ssl/cert.pem \
      -v /etc/your_certificate_directory/privkey.pem:/etc/rancher/ssl/key.pem \
      <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG> \
      --no-cacerts
    ```

## 三、通过HTTP L7层代理访问Rancher server

1. 配置七层负载均衡器

    默认情况下，rancher容器会将80端口上的请求重定向到443端口上。如果rancher server通过负载均衡器来代理，这个时候请求是通过负载均衡器发送给rancher server，而并非客户端直接访问rancher server。在非全局`https`的环境中，如果以外部负载均衡器作为ssl终止，这个时候通过负载均衡器的`https`请求将需要被反向代理到rancher server http(80)上。在负载均衡器上配置`X-Forwarded-Proto: https`参数，rancher server http(80)上收到负载均衡器的请求后，就不会再重定向到https(443)上。

    负载均衡器或代理必须支持以下参数:

    - **WebSocket** 连接
    - **SPDY**/**HTTP/2**协议
    - 传递/设置以下headers:

    | Header              | Value             | 描述            |
    |---------------------|--------------------------|:------------|
    | `Host`              | 传递给Rancher的主机名| 识别客户端请求的主机名。      |
    | `X-Forwarded-Proto` | `https`       | 识别客户端用于连接负载均衡器的协议。**注意：**如果存在此标头，`rancher/rancher`不会将HTTP重定向到HTTPS。     |
    | `X-Forwarded-Port`  | Port used to reach Rancher.   | 识别客户端用于连接负载均衡器的端口。      |
    | `X-Forwarded-For`   | IP of the client connection.   | 识别客户端的原始IP地址。            |

    >nginx配置示例

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
            listen 443 ssl http2; # 如果是升级或者全新安装v2.2.2,需要禁止http2
            server_name FQDN;
            ssl_certificate <更换为自己的证书>;
            ssl_certificate_key <更换为自己的证书私钥>;

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

1. 使用自己的自签名证书

    采用外部七层负载均衡器来做代理，那么只需要把证书放在外部七层负载均衡器上，如果是自签名证书，则需要把CA文件映射到rancher server容器中。如果没有自签名证书，可一键生成[自签名ssl证书]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/self-signed-ssl/)。

    > **先决条件:**
    > - 使用OpenSSL或其他方法创建自签名证书。\
    > - 这里的证书不需要进行`base64`加密。\
    > - 证书文件必须是[PEM]({{< baseurl >}}/rancher/v2.x/cn/installation/single-node-install/#我如何知道我的证书是否为pem格式)格式。\
    > - 在你的证书文件中，包含链中的所有中间证书。有关示例，请参考[SSL常见问题/故障排除]({{< baseurl >}}/rancher/v2.x/cn/installation/   single-node-install/#如果我想添加我的中间证书-证书的顺序是什么)。

    运行Rancher server容器时候，需要把自签名CA证书映射到Rancher容器中:

    ```bash
    docker run -d --restart=unless-stopped \
      -p 80:80 -p 443:443 \
      -v /var/log/rancher/auditlog:/var/log/auditlog \
      -e AUDIT_LEVEL=3 \
      -v <主机路径>:/var/lib/rancher/ \
      -v /etc/<CERT_DIRECTORY>/<CA_CERTS.pem>:/etc/rancher/ssl/cacerts.pem \
     <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
    ```

1. 使用权威CA机构颁发的证书

    如果你公开发布你的应用，理想情况下应该使用由权威CA机构颁发的证书。如果你使用由权威CA机构颁发的证书，则无需在Rancher容器中安装你的CA证书，只需运行下面的基本安装命令即可:

    > **先决条件:**
    >1.证书必须是`PEM格式`,`PEM`只是一种证书类型，并不是说文件必须是PEM为后缀，具体可以查看[证书类型]({{< baseurl >}}/rancher/v2.x/cn/       install-prepare/self-signed-ssl/)。\
    >2.确保容器包含你的证书文件和密钥文件。由于你的证书是由认可的CA签署的，因此不需要安装额外的CA证书文件。\
    >3.给容器添加`--no-cacerts`参数禁止Rancher生成默认CA证书。\
    >4.这里的证书不需要进行`base64`加密。

    ```bash
    docker run -d --restart=unless-stopped \
      -p 80:80 -p 443:443 \
      -v <主机路径>:/var/lib/rancher/ \
      -v /root/var/log/auditlog:/var/log/auditlog \
      -e AUDIT_LEVEL=3 \
        <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG> \
      --no-cacerts
    ```

## 四、(可选)为Agent Pod添加主机别名(/etc/hosts)

如果你没有内部DNS服务器而是通过添加`/etc/hosts`主机别名的方式指定的Rancher server域名，那么不管通过哪种方式(自定义、导入、Host驱动等)创建K8S集群，K8S集群运行起来之后，因为`cattle-cluster-agent Pod`和`cattle-node-agent`无法通过DNS记录找到`Rancher server`,最终导致无法通信。

### 解决方法

可以通过给`cattle-cluster-agent Pod`和`cattle-node-agent`添加主机别名(/etc/hosts)，让其可以正常通信`(前提是IP地址可以互通)`。

1. cattle-cluster-agent pod

    ```bash
    #指定kubectl配置文件
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml

    kubectl --kubeconfig=kube_configxxx.yml -n cattle-system \
    patch deployments cattle-cluster-agent --patch '{
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
    #指定kubectl配置文件
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml

    kubectl --kubeconfig=kube_configxxx.yml -n cattle-system \
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

    > **注意**
    >1、替换其中的域名和IP \
    >2、别忘记json中的引号。

## 五、FAQ和故障排除

[FAQ]({{< baseurl >}}/rancher/v2.x/cn/faq/)中整理了常见的问题与解决方法。
