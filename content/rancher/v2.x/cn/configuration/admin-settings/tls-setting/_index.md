---
title: 12 - TLS 设置
weight: 12
---

> 自v2.1.7起可用

在Rancher v2.1.7中，默认TLS配置为仅接受TLS 1.2和安全TLS密码套件,不支持TLS 1.3和TLS 1.3独占密码套件。

## TLS设置

| 参数                     | 描述              | 默认                                                         | 可用选项                                                     |
| :----------------------- | :---------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `CATTLE_TLS_MIN_VERSION` | 最低TLS版本       | `1.2`                                                        | `1.0`，`1.1`，`1.2`                                          |
| `CATTLE_TLS_CIPHERS`     | 允许的TLS密码套件 | `TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,` `TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,` `TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,` `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,` `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,` `TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305` | 请参阅[Golang tls常量](https://golang.org/pkg/crypto/tls/#pkg-constants) |

## 旧版配置

如果您需要以与Rancher v2.1.7之前相同的方式配置TLS，请使用以下设置：

| 参数                     | 遗产价值                                                     |
| :----------------------- | :----------------------------------------------------------- |
| `CATTLE_TLS_MIN_VERSION` | `1.0`                                                        |
| `CATTLE_TLS_CIPHERS`     | `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,` `TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,` `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,` `TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,` `TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,` `TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,` `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,` `TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA,` `TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,` `TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA,` `TLS_RSA_WITH_AES_128_GCM_SHA256,` `TLS_RSA_WITH_AES_256_GCM_SHA384,` `TLS_RSA_WITH_AES_128_CBC_SHA,` `TLS_RSA_WITH_AES_256_CBC_SHA,` `TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA,` `TLS_RSA_WITH_3DES_EDE_CBC_SHA` |

## 配置TLS设置

通过将环境变量传递给Rancher服务器容器来启用和配置审核日志。请参阅以下内容以启用安装。

### 1. 单节点安装

```bash
docker run -d --restart=unless-stopped \
-p 80:80 -p 443:443 \
 -v <主机路径>:/var/lib/rancher/ \
-e CATTLE_TLS_MIN_VERSION="1.0" \
rancher/rancher:latest
```

### 2. HA安装

```bash
--set 'extraEnv[0].name=CATTLE_TLS_MIN_VERSION'
--set 'extraEnv[0].value=1.0'
```