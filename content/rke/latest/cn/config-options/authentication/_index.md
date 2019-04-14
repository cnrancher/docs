---
title: 8 - 认证
weight: 8
---

RKE支持x509身份验证策略，您可以定义要添加到Kubernetes API服务器PKI证书的SAN列表（备用域名或者ip）。

> 例如，这允许您通过负载均衡器而不是单个节点连接到Kubernetes集群API服务器。

```yaml
authentication:
    strategy: x509
    sans:
      - "10.18.160.10"
      - "my-loadbalancer-1234567890.us-west-2.elb.amazonaws.com"
```

RKE也支持webhook身份验证策略，您可以通过`|`在配置中使用`分隔符`来启用`x509和webhook策略`。应提供webhook配置文件的内容，有关文件格式的信息，请参阅[Kubernetes webhook文档](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication) 。此外，可以设置webhook身份验证响应的缓存超时。

```yaml
authentication:
    strategy: x509|webhook
    webhook:
      config_file: "...."
      cache_timeout: 5s
```
