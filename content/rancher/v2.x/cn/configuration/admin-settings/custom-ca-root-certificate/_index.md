---
title: 8 - 自定义CA根证书
weight: 8
---
如果Rancher访问的应用使用的是`自签名ssl`，那么如果不给Rancher指定`自签名CA根证书`，则会显示以下错误：`x509: certificate signed by unknown authority`。

要正常验证应用证书，需要将`CA根证书`添加到Rancher。由于Rancher是用Go编写的，我们可以使用环境变量`SSL_CERT_DIR`指向容器中`CA根证书`所在的目录。并且在启动Rancher容器时，可以使用`-v host-source-directory:container-destination-directory`挂载主机上`CA根证书目录`到容器中。

哪些应用需要自定义CA根证书？

- 自定义应用商店(使用ssl自签名证书)
- 第三方登录认证平台(使用ssl自签名证书)
- 使用节点驱动访问托管/云API(使用CA证书进行认证)

## 配置方法

### 单节点

- `-v`: 映射宿主机目录到容器
- `-e`: 定义环境变量，配置容器CA根证书目录

```bash
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v <主机路径>:/var/lib/rancher/ \
  -v /host/certs:/container/certs \
  -e SSL_CERT_DIR="/container/certs" \
  rancher/rancher:stable (或者rancher/rancher:latest)
```

### HA

将您的CA证书以pem格式复制到名为`ca-additional.pem`的文件中，并用`kubectl`在命名空间`cattle-system`中创建`tls-ca-additional`密文。

```plain
kubectl --kubeconfig=kube_configxxx.yml create  namespace cattle-system
kubectl --kubeconfig=kube_configxxx.yml -n cattle-system create secret generic tls-ca-additional --from-file=ca-additional.pem
```
然后，在Helm部署Rancher server时，添加以下参数

```plain
--set additionalTrustedCAs=true
```

