---
title: 2 - 添加TLS secret
weight: 2
---

Kubernetes将为Rancher创建所有对象和服务，但在我们使用证书和secret填充命名空间中的tls-rancher-ingress secret之前，它将无法使用cattle-system

## 权威CA机构颁发的证书

将`服务证书`和`CA中间证书链`合并到一个名为`tls.crt`的文件中,将`私钥`复制到名为`tls.key`的文件中。

使用`kubectl`与`tls`类型来创建`secrets`.

```
kubectl -n cattle-system create secret tls tls-rancher-ingress \
  --cert=./tls.crt \
  --key=./tls.key
```

## 私有CA签名证书(可选)

如果你使用的是私有CA，则需要传递CA文件给Rancher。

将CA证书复制到名为`cacerts.pem`的文件中，并用`kubectl`在命名空间`cattle-system`中创建`tls-ca`secret。

```
kubectl -n cattle-system create secret generic tls-ca \
  --from-file=cacerts.pem
```
