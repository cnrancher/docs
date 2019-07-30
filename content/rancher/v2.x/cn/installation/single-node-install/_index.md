---
title: 3 - 单节点安装
weight: 3
---

对于开发环境，我们推荐直接在主机上通过`docker run`的形式运行Rancher Server容器。可能有的主机无法直接通过公网IP来访问主机，需要通过代理去访问，这种场景请参考[使用外部负载平衡器进行单一节点安装](./single-node-install-external-lb/)。

## 一、Linux主机要求

- [基础环境配置]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/basic-environment-configuration/)
- [端口需求]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/references/)

## 二、配置SSL证书并安装Rancher

出于安全考虑，使用Rancher时需要SSL进行加密。SSL可以保护所有Rancher网络通信，例如登录或与集群交互。

> **注意:**
1、[离线安装]({{< baseurl >}}/rancher/v2.x/cn/installation/air-gap-installation/)?\
2、[API审计日志]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/api-auditing/)?\
3、[代理上网]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/proxy-configuration/)?\
4、[自定义CA根证书]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/custom-ca-root-certificate/)?

### 方案A-使用默认自签名证书

默认情况下，Rancher会自动生成一个用于加密的自签名证书。从您的Linux主机运行Docker命令来安装Rancher，而不需要任何其他参数:

```bash
docker run -d --restart=unless-stopped \
-p 80:80 -p 443:443 \
-v <主机路径>:/var/lib/rancher/ \
-v /root/var/log/auditlog:/var/log/auditlog \
-e AUDIT_LEVEL=3 \
rancher/rancher:stable (或者rancher/rancher:latest)
```

### 方案B-使用您自己的自签名证书

Rancher安装可以使用自己生成的自签名证书，如果没有自签名证书，可一键生成[自签名ssl证书]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/self-signed-ssl/)。

> **先决条件:**
> - 使用OpenSSL或其他方法创建自签名证书。\
> - 这里的证书不需要进行`base64`加密。\
> - 证书文件必须是[PEM]({{< baseurl >}}/rancher/v2.x/cn/installation/single-node-install/#我如何知道我的证书是否为pem格式)格式。\
> - 在您的证书文件中，包含链中的所有中间证书。有关示例，请参考[SSL常见问题/故障排除]({{< baseurl >}}/rancher/v2.x/cn/installation/single-node-install/#如果我想添加我的中间证书-证书的顺序是什么)。

您的Rancher安装可以使用您提供的自签名证书来加密通信。创建证书后，运行docker命令时把证书文件映射到容器中。

```bash
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v <主机路径>:/var/lib/rancher/ \
  -v /var/log/rancher/auditlog:/var/log/auditlog \
  -e AUDIT_LEVEL=3 \
  -v /etc/<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
  -v /etc/<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
  -v /etc/<CERT_DIRECTORY>/<CA_CERTS.pem>:/etc/rancher/ssl/cacerts.pem \
  rancher/rancher:stable (或者rancher/rancher:latest)
```

### 方案C-使用权威CA机构颁发的证书

如果您公开发布您的应用，理想情况下应该使用由权威CA机构颁发的证书。

> **先决条件:**
>1.证书必须是`PEM格式`,`PEM`只是一种证书类型，并不是说文件必须是PEM为后缀，具体可以查看[证书类型]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/self-signed-ssl/)。\
>2.确保容器包含您的证书文件和密钥文件。由于您的证书是由认可的CA签署的，因此不需要安装额外的CA证书文件。\
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
  rancher/rancher:stable (或者rancher/rancher:latest) --no-cacerts
```

## 三、FAQ和故障排除

[FAQ]({{< baseurl >}}/rancher/v2.x/cn/faq/)中整理了常见的问题与解决方法。
