---
title: 外部etcd
weight: 1
---

默认情况下，RKE将部署`etcd服务`，RKE也支持使用`外部etcd`,但RKE仅支持连接到启用了`TLS的etcd`。

> **注意:** RKE不支持将外部`etcd服务`与具有`etcd`角色的节点混合使用。

```yaml
services:
  etcd:
    path: /etcdcluster
    external_urls:
      - https://etcd-example.com:2379
    ca_cert: |-
      -----BEGIN CERTIFICATE-----
      xxxxxxxxxx
      -----END CERTIFICATE-----
    cert: |-
      -----BEGIN CERTIFICATE-----
      xxxxxxxxxx
      -----END CERTIFICATE-----
    key: |-
      -----BEGIN PRIVATE KEY-----
      xxxxxxxxxx
      -----END PRIVATE KEY-----
```

## 外部etcd选项

- Path

`path`定义etcd集群在端点上的位置。

- External URLs

`external_urls`是etcd集群的对外暴露的端点，etcd集群可以有多个端点。

- CA Cert/Cert/KEY

用于验证和访问etcd服务的证书和私钥。
