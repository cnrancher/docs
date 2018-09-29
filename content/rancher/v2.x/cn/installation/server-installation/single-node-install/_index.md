---
title: 1 - 独立容器安装
weight: 1
---

对于开发环境，我们推荐直接在主机上通过`docker run`的形式运行Rancher server容器。可能有的主机无法直接通过公网IP来访问主机，需要通过代理去访问，这种场景请参考[使用外部负载平衡器进行单一节点安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/single-node-install-external-lb/)。

{{% accordion id="1" label="一、Linux主机要求" %}}

1、[基础环境配置]({{< baseurl >}}/rancher/v2.x/cn/installation/basic-environment-configuration/)

2、[端口需求]({{< baseurl >}}/rancher/v2.x/cn/installation/references/)

{{% /accordion %}}
{{% accordion id="2" label="二、安装Rancher并配置SSL证书" %}}

出于安全考虑，使用Rancher时需要SSL进行加密。SSL可以保护所有Rancher网络通信，例如登录或与集群交互。

> **注意** 如果你正在访问此页面以完成[离线安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/air-gap-installation/)，在运行安装命令时，必须在Rancher镜像前面加上你私有仓库的地址，替换`<REGISTRY.DOMAIN.COM:PORT>`为你的私有仓库地址。\
> 例如:`<REGISTRY.DOMAIN.COM:PORT>/rancher/rancher:latest` \
> 如果想开启API审计日志功能，请访问[API审计日志]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/api-auditing/)。

### 1、方案A-使用默认自签名证书

默认情况下，Rancher会自动生成一个用于加密的自签名证书。

**使用默认证书安装Rancher:**

从你的Linux主机运行Docker命令来安装Rancher，而不需要任何其他参数:

```bash
docker run -d --restart=unless-stopped \
-p 80:80 -p 443:443 \
rancher/rancher:latest
```

### 2、方案B-使用你自己的自签名证书

Rancher安装可以使用自己生成的自签名证书。

> **先决条件** 创建一个自签名证书。
> - 这里的证书不需要进行`base64`加密;
> - 证书文件必须是[PEM]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/single-node-install/#我如何知道我的证书是否为pem格式)格式;
> - 在你的证书文件中，包含链中的所有中间证书。有关示例，请参考[SSL常见问题/故障排除]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/single-node-install/#如果我想添加我的中间证书-证书的顺序是什么);

**使用自己生成的自签名证书安装Rancher:**

你的Rancher安装可以使用你提供的自签名证书来加密通信。创建证书后，运行docker命令时把证书文件映射到容器中。

```bash
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /etc/<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
  -v /etc/<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
  -v /etc/<CERT_DIRECTORY>/<CA_CERTS.pem>:/etc/rancher/ssl/cacerts.pem \
  rancher/rancher:latest
```

### 3、方案C-使用权威CA机构颁发的证书

如果你公开发布你的应用，理想情况下应该使用由权威CA机构颁发的证书。

> **先决条件:**1.证书必须是`PEM格式`,`PEM`只是一种证书类型，并不是说文件必须是PEM为后缀，具体可以查看[证书类型]({{< baseurl >}}/rancher/v2.x/cn/installation/self-signed-ssl/)；\
>2.确保容器包含你的证书文件和密钥文件。由于你的证书是由认可的CA签署的，因此不需要安装额外的CA证书文件;\
>3.给容器添加`--no-cacerts`参数禁止Rancher生成默认CA证书；\
>4.这里的证书不需要进行`base64`加密;

获取证书后，运行Docker命令以部署Rancher，同时指向证书文件。

```bash
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /etc/your_certificate_directory/fullchain.pem:/etc/rancher/ssl/cert.pem \
  -v /etc/your_certificate_directory/privkey.pem:/etc/rancher/ssl/key.pem \
  rancher/rancher:latest --no-cacerts
```

默认情况下，Rancher会在安装时自动为自己生成自签名CA证书。但是，由于你提供了自己的证书，因此必须禁用Rancher自动生成的CA证书。

**删除默认证书:**

1. 登录Rancher。
2. 选择 **设置** > **cacerts**。
3. 选择`Edit`并删除内容。然后点击`Save`。

### 4、方案D-Let’s Encrypt 证书

Rancher支持Let’s Encrypt 证书。Let’s Encrypt 使用一个`http-01 challenge`来验证你是否是该域名的所有者。你可以通过将想要用于Rancher访问的主机名(例如，`rancher.mydomain.com`)指向正Rancher server主机IP，以此来确认你是否是该域名的所有者。你可以通过在DNS中创建A记录来将主机名绑定到IP地址。

> **先决条件:**1.Let's Encrypt是一项在线互联网服务，因此不能用于内部/离线网络。**注意：有的let's证书不支持HTTP2协议，请确认你的证书是否支持。**\
>2.在你的DNS中创建一条记录，将你的Linux主机IP地址绑定到你想要用于Rancher访问的主机名(例如:`rancher.mydomain.com`)。\
>3.在你的Linux主机上打开`TCP/80`端口,Let's Encrypt http-01检查可能来自任意的源IP地址，因此端口`TCP/80`必须对所有IP地址开放。

**使用Let's Encrypt证书安装Rancher:**

从你的Linux主机运行以下命令。

运行Docker命令。

```bash
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  rancher/rancher:latest \
  --acme-domain rancher.mydomain.com
```

>**注意:Let’s Encrypt 平台对证书的申请和销毁有一定频率限制。有关更多信息，请参考[Let’s Encrypt documentation on rate limits](https://letsencrypt.org/docs/rate-limits/)。

{{% /accordion %}}
{{% accordion id="3" label="三、下一步" %}}

你有几个选择:

- 创建Rancher server的备份: [单节点备份和恢复]({{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/backups/single-node-backups/)。
- 创建一个Kubernetes集群: [创建一个集群]({{< baseurl >}}/rancher/v2.x/cn/configuration/clusters/creating-a-cluster/)。

{{% /accordion %}}
{{% accordion id="4" label="四、FAQ和故障排除" %}}

[FAQ]({{< baseurl >}}/rancher/v2.x/cn/faq/)中整理了常见的问题与解决方法。

{{% /accordion %}}