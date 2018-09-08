---
title: 2 - 独立容器安装+外部负载平衡
weight: 2
---

对于开发环境，我们推荐直接在主机上通过`docker run`的形式运行Rancher server容器。如果主机可以通过公网IP直接访问，可以参考[单节点安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/single-node-install)。

##  一、Linux主机要求

配置一台Linux主机来运行Rancher server。

### 1、操作系统

- Ubuntu 16.04(64位)
- Centos/RedHat Linux 7.5+(64位)
- RancherOS 1.3.0+(64位)

### 2、硬件

硬件需求根据Rancher部署的规模进行扩展。根据需求配置每个节点。

| 部署大小 | 集群(个)  | 节点(个) | vCPU                                            | 内存 |
| -------- | --------- | -------- |     ------------------------------------------- | ---- |
| 小       | 不超过10  | 最多50   | 2C                                              | 4GB  |
| 中       | 不超过100 | 最多500  | 8C                                              | 32GB |
| 大       | 超过100   | 超过500  | [联系Rancher](https://www.cnrancher.com/contact/) |      |

### 3、软件

- Docker

    > **注意:**如果你使用的是RancherOS，请确保你将Docker引擎切换为受支持的版本`sudo ros engine switch docker-17.03.2-ce`

    **支持的Docker版本**

  - `1.12.6`
  - `1.13.1`
  - `17.03.2`

    [Docker安装说明](https://docs.docker.com/install/)

    > **注意:** 1.该`rancher/rancher`镜像托管在[DockerHub上](https://hub.docker.com/r/rancher/rancher/tags/)。如果你无法访问DockerHub，或者离线环境下安装Rancher，请查阅[离线安装](/docs/rancher/v2.x/cn/installation/server-installation/air-gap-installation/)。\
    >2.更多Rancher server tag列表，，请查阅[Rancher server tags](/docs/rancher/v2.x/cn/installation/server-tags/)。

### 4、端口

下图描述了Rancher的基本端口要求。有关全面列表，请查阅[端口要求](/docs/rancher/v2.x/cn/installation/references/)。

![基本端口要求](/docs/img/rancher/port-communications.png)

## 二、安装Rancher并配置SSL证书

出于安全考虑，使用Rancher时需要SSL进行加密。SSL可以保护所有Rancher网络通信，例如登录或与集群交互。

> **注意:** 1.如果你正在访问此页面以完成[离线安装](/docs/rancher/v2.x/cn/installation/server-installation/air-gap-installation/)，在运行安装命令时，必须在Rancher镜像前面加上你私有仓库的地址，替换`<REGISTRY.DOMAIN.COM:PORT>`为你的私有仓库地址。\
> 例如: <REGISTRY.DOMAIN.COM:PORT>/rancher/rancher:latest \
>2.如果想开启API审计日志功能，请访问[API审计日志](/docs/rancher/v2.x/cn/installation/server-installation/api-auditing/)。

### 1、方案A-使用你自己生成的自签名证书

如果你选择使用自签名证书来加密通信，则必须将证书安装在负载均衡器上，并且将CA证书放置于Rancher容器中。

> **先决条件:**1.创建一个自签名证书;\
>2.证书文件必须是**PEM**格式;\
>3.这里的证书不需要进行`base64`加密;

运行Rancher server容器时候，需要把自签名CA证书映射到Rancher容器中:

```
docker run -d --restart=unless-stopped \
-p 80:80 -p 443:443 \
-v /etc/your_certificate_directory/cacerts.pem:/etc/rancher/ssl/cacerts.pem \
rancher/rancher:latest
```

### 2、方案B-使用权威CA机构颁发的证书

如果你公开发布你的应用，理想情况下应该使用由权威CA机构颁发的证书。

> **先决条件:**1.证书必须是`PEM格式`,`PEM`只是一种证书类型，并不是说文件必须是PEM为后缀，具体可以查看[证书类型](/docs/rancher/v2.x/cn/installation/self-signed-ssl/)；\
>2.这里的证书不需要进行`base64`加密;\
>3.给容器添加`--no-cacerts`参数禁止Rancher生成默认CA证书；

如果你使用由权威CA机构颁发的证书，则无需在Rancher容器中安装你的CA证书，只需运行下面的基本安装命令即可:

```
docker run -d --restart=unless-stopped \
-p 80:80 -p 443:443 \
rancher/rancher:latest --no-cacerts
```

## 三、配置七层负载均衡器

默认情况下，通过`docker run`运行的Rancher server容器会自动把端口80重定向到443，但是通过负载均衡器来代理Rancher server容器后，不再需要将Rancher server容器端口从80重定向到443。通过在负载均衡器上配置`X-Forwarded-Proto: https`参数后，Rancher server容器端口重定向功能将自动被禁用。

负载均衡器或代理必须配置为支持以下内容:

- **WebSocket** 连接
- **SPDY**/**HTTP/2** 协议
- 传递/设置以下headers:

| Header       | Value       | 描述     |
|-----------|-------------|:-----------|
| `Host`              | 传递给Rancher的主机名| 识别客户端请求的主机名。          |
| `X-Forwarded-Proto` | `https`     | 识别客户端用于连接负载均衡器的协议。**注意：**如果存在此标头，`rancher/ancher`不会将HTTP重定向到HTTPS。 |
| `X-Forwarded-Port`  | Port used to reach Rancher.  | 识别客户端用于连接负载均衡器的协议。   |
| `X-Forwarded-For`   | IP of the client connection.  | 识别客户端的原始IP地址。  |

### 四、Nginx 配置文件示例

此Nginx配置文件在Nginx version 1.13 (mainline)和1.14(stable)通过测试

```
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

## 五、删除默认CA证书

>注意: 此操作仅适用于使用权威CA机构颁发的证书，如果你使用的是自签名证书，请不要进行此操作。

默认情况下，Rancher会在安装时自动为自己生成自签名CA证书。但是，由于你提供了自己的证书，因此必须禁用Rancher自动生成的CA证书。

**删除默认证书:**

1. 登录Rancher。
2. 选择 **设置** > **cacerts**。
3. 选择`Edit`并删除内容。然后点击`Save`。

## 六、下一步？

你有几个选择:

- 创建Rancher server的备份:[单节点备份和恢复](/docs/rancher/v2.x/cn/backups-and-restoration/backups/single-node-backups/)。
- 创建一个Kubernetes集群:[创建一个集群](/docs/rancher/v2.x/cn/configuration/clusters/creating-a-cluster/)。

## 七、FAQ和故障排除

[FAQ](/docs/rancher/v2.x/cn/faq/)中整理了常见的问题与解决方法。
