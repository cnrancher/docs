---
title: 1 - Chart设置参数
weight: 1
---

## 常见设置

| Option | Default Value | Description |
| --- | --- | --- |
| `hostname` | " " | `string` - 为访问Rancher设置的域名 |
| `ingress.tls.source` | "rancher" | `string` - 证书获取方式:`rancher, letsEncrypt, secret` |
| `letsEncrypt.email` | " " | `string` - 邮件地址 |
| `letsEncrypt.environment` | "production" | `string` - Valid options: "staging, production" |
| `privateCA` | false | `bool` - 如果是自签名证书，则设置为`true` |

## 高级设置

| Option | Default Value | Description |
| --- | --- | --- |
| `debug` | false | `bool` - 设置rancher server日志等级 |
| `imagePullSecrets` | [] | `list` - 私有仓库访问密文证书 |
| `proxy` | "" | `string` - string - HTTP[S] proxy server for Rancher |
| `noProxy` | "localhost,127.0.0.1" | `string` - comma seperated list of hostnames or ip address not to use the proxy |
| `resources` | {} | `map` - rancher pod resource requests & limits |
| `rancherImage` | "rancher/rancher" | `string` - rancher image source |
| `rancherImageTag` | same as chart version | `string` - rancher/rancher image tag |
| `tls` | "ingress" | `string` - Where to terminate SSL. - "ingress, external"

### HTTP Proxy

在某些需要代理上网的环境下，通过`proxy`设置代理服务器用以获取网络资源。

访问cluster IP、Pod IP、以及其他内部网络IP，可以通过`noProxy`排除代理列表外：

```
--set proxy="http://<username>:<password>@<proxy_url>:<proxy_port>/"
--set noProxy="127.0.0.1,localhost,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"
```

### 私有镜像仓库

可以为Rancher配置私有镜像仓库地址，用以获取私有镜像仓库的镜像。

#### Images

Populate your private registry with Rancher images.

你可以从[Releases](https://github.com/rancher/rancher/releases/latest)页面获取到最新Rancher可用镜像列表，下载镜像并同步到私有仓库。

#### 创建Registry Secret

用`kubectl`在`cattle-system`命名空间中创建docker-registry secret 。

```
kubectl -n cattle-system create secret docker-registry regcred \
  --docker-server="reg.example.com:5000" \
  --docker-email=<email>
```

#### 仓库设置选项

添加`rancherImage`指向私有镜像仓库镜像和`imagePullSecrets`。

```
--set rancherImage=reg.example.com:5000/rancher/rancher \
--set imagePullSecrets[0].name=regcred
```

### 外部TLS Termination

If you wish to terminate the SSL/TLS on a load-balancer external to the Rancher cluster (ingress), use the `--tls=external` option and point your load balancer at port http 80 on all of the rancher cluster nodes.

> NOTE: If you are using a Private CA signed cert, add `--set privateCA=true` and see [Adding TLS Secrets - Private CA Signed - Additional Steps](../tls-secrets/#private-ca-signed---additional-steps) to add the CA cert for Rancher.

Your load balancer must support long lived websocket connections and will need to insert proxy headers so Rancher can route links correctly.

> NOTE: The `tls=external` option will expose the Rancher interface on http port 80.  Clients that are allowed to connect directly to the Rancher cluster will not be encrypted. We recommend that you restrict direct access at the network level to just your load balancer.

#### Required headers

* `Host`
* `X-Forwarded-Proto`
* `X-Forwarded-Port`
* `X-Forwarded-For`

#### Health checks

Rancher will respond `200` to health checks on the `/healthz` endpoint.
