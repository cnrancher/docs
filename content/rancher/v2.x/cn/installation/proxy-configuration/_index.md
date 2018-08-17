---
title: 6、Rancher HTTP代理配置
weight: 4
---

如果你的环境需要通过代理才可以连接互联网，那么需要配置`HTTP_PROXY`或者`HTTPS_PROXY`代理,Rancher才能获取网络资源。

| 环境变量    | 作用                   |
| ----------- | ---------------------|
| HTTP_PROXY  | 访问HTTP链接设置的代理  |
| HTTPS_PROXY | 访问HTTPS链接设置的代理 |
| NO_PROXY    | 不通过代理访问的地址    |

> 代理变量需要大写

## Rancher单节点安装

单容器安装Rancher时，可将代理参数以环境变量的形式传递给Rancher容器。比如:http://192.168.0.1:3128 是代理服务器地址，并且要求访问网络 `192.168.10.0/24`和`example.com`域名下的每个子域名时排除使用代理

```bash
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -e HTTP_PROXY="http://192.168.10.1:3128" \
  -e HTTPS_PROXY="http://192.168.10.1:3128" \
  -e NO_PROXY="localhost,127.0.0.1,0.0.0.0,192.168.10.0/24,example.com" \
  rancher/rancher:latest
```

## Rancher HA安装

在Rancher HA安装时，需要将代理参数添加到RKE配置文件模板。

```yaml
...
---
  kind: Deployment
  apiVersion: extensions/v1beta1
  metadata:
    namespace: cattle-system
    name: cattle
  spec:
    replicas: 1
    template:
      metadata:
        labels:
          app: cattle
      spec:
        serviceAccountName: cattle-admin
        containers:
        - image: rancher/rancher:latest
          imagePullPolicy: Always
          name: cattle-server
          env:
          - name: HTTP_PROXY
            value: "http://192.168.10.1:3128"
          - name: HTTPS_PROXY
            value: "http://192.168.10.1:3128"
          - name: NO_PROXY
            value: "localhost,127.0.0.1,0.0.0.0,10.43.0.0/16,192.168.10.0/24,example.com"
          ports:
...
```