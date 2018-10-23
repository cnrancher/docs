---
title: 1 - 独立容器安装+外部七层负载平衡
weight: 1
---

对于开发环境，我们推荐直接在主机上通过`docker run`的形式运行Rancher server容器。如果主机可以通过公网IP直接访问，可以参考[单节点安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/single-node-install)。

### 一、说明: 为什么使用外部七层负载平衡而不使用外部四层负载平衡？

因为SSL需要运行在http七层协议上，所以如果外部负载均衡采用四层协议，那么SSL证书就只能安装在Rnacher server上，这种架构将达不到采用负载均衡器预期的效果。

### 二、Linux主机要求

- [基础环境配置]({{< baseurl >}}/rancher/v2.x/cn/installation/basic-environment-configuration/)
- [端口需求]({{< baseurl >}}/rancher/v2.x/cn/installation/references/)

### 三、安装Rancher并配置SSL证书

出于安全考虑，使用Rancher时需要SSL进行加密。SSL可以保护所有Rancher网络通信，例如登录或与集群交互。

> **注意:**
> 1、需要[离线安装？]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/air-gap-installation/) \
> 2、需要开启[API审计日志？]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/api-auditing/) \
> 3、需要[代理上网?]({{< baseurl >}}/rancher/v2.x/cn/installation/proxy-configuration/)

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
>1、证书必须是`PEM格式`,`PEM`只是一种证书类型，并不是说文件必须是PEM为后缀，具体可以查看[证书类型]({{< baseurl >}}/rancher/v2.x/cn/installation/self-signed-ssl/)；\
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

### 四、配置七层负载均衡器

默认情况下，通过`docker run`运行的Rancher server容器会自动把端口80重定向到443，但是通过负载均衡器来代理Rancher server容器后，不再需要将Rancher server容器端口从80重定向到443。通过在负载均衡器上配置`X-Forwarded-Proto: https`参数后，Rancher server容器端口重定向功能将自动被禁用。

负载均衡器或代理必须配置为支持以下内容:

- **WebSocket**连接
- **SPDY**/**HTTP/2**协议
- 传递/设置以下headers:

    | Header       | Value       | 描述     |
    |-----------|-------------|:-----------|
    | `Host`              | 传递给Rancher的主机名| 识别客户端请求的主机名。          |
    | `X-Forwarded-Proto` | `https`     | 识别客户端用于连接负载均衡器的协议。**注意：**如果存在此标头，`rancher/ancher`不会将HTTP重定向到HTTPS。 |
    | `X-Forwarded-Port`  | Port used to reach Rancher.  | 识别客户端用于连接负载均衡器的协议。   |
    | `X-Forwarded-For`   | IP of the client connection.  | 识别客户端的原始IP地址。  |

### 五、Nginx 配置文件示例

此Nginx配置文件在Nginx version 1.13 (mainline)和1.14(stable)通过测试

```bash
upstream rancher {
    server rancher-server:80;
}

map $http_upgrade $connection_upgrade {
    default Upgrade;
    ''      close;
}

server {
    listen 443 ssl http2;
    server_name rancher.yourdomain.com;
    ssl_certificate /etc/your_certificate_directory/fullchain.pem;
    ssl_certificate_key /etc/your_certificate_directory/privkey.pem;

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
    }
}

server {
    listen 80;
    server_name rancher.yourdomain.com;
    return 301 https://$server_name$request_uri;
}
```

### 六、下一步？

你有几个选择:

- 创建Rancher server的备份:[单节点备份和恢复]({{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/backups/single-node-backups/)。
- 创建一个Kubernetes集群:[创建一个集群]({{< baseurl >}}/rancher/v2.x/cn/configuration/clusters/creating-a-cluster/)。

### 七、FAQ和故障排除

[FAQ]({{< baseurl >}}/rancher/v2.x/cn/faq/)中整理了常见的问题与解决方法。
