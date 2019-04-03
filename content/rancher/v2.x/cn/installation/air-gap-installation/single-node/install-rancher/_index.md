---
title: 3 - 安装Rancher
weight: 3
---

出于安全考虑，使用Rancher时需要SSL进行加密。SSL可以保护所有Rancher网络通信，例如登录或与集群交互。

> 1. [自定义CA证书]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/custom-ca-root-certificate/)
> 2. [开启审计日志]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/api-auditing/)
> 3. [TLS设置]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/tls-setting/)

## 一、默认自签名证书安装

默认情况下，Rancher会自动生成一个用于加密的自签名证书。从你的Linux主机运行Docker命令来安装Rancher，而不需要任何其他参数:

```bash
docker run -d --restart=unless-stopped \
-p 80:80 -p 443:443 \
-v <主机路径>:/var/lib/rancher/ \
-v /root/var/log/auditlog:/var/log/auditlog \
-e AUDIT_LEVEL=3 \
<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
```

## 二、使用自己的自签名证书

Rancher安装可以使用自己生成的自签名证书。

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

## 三、使用权威CA机构颁发的证书

如果你公开发布你的应用，理想情况下应该使用由权威CA机构颁发的证书。

> **先决条件:**
>1.证书必须是`PEM格式`,`PEM`只是一种证书类型，并不是说文件必须是PEM为后缀，具体可以查看[证书类型]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/self-signed-ssl/)。\
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

## 四、(可选)为Agent Pod添加主机别名(/etc/hosts)

如果你没有内部DNS服务器而是通过添加`/etc/hosts`主机别名的方式指定的Rancher server域名，那么不管通过哪种方式(自定义、导入、Host驱动等)创建K8S集群，K8S集群运行起来之后，因为`cattle-cluster-agent Pod`和`cattle-node-agent`无法通过DNS记录找到`Rancher server`,最终导致无法通信。

### 解决方法

可以通过给`cattle-cluster-agent Pod`和`cattle-node-agent`添加主机别名(/etc/hosts)，让其可以正常通信`(前提是IP地址可以互通)`。

1. cattle-cluster-agent pod

    ```bash
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml #指定kubectl配置文件
    kubectl --kubeconfig=kube_configxxx.yml -n cattle-system patch  deployments cattle-cluster-agent --patch '{
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
    kubectl --kubeconfig=kube_configxxx.yml -n cattle-system patch  daemonsets cattle-node-agent --patch '{
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
