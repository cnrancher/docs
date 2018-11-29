---
title: 2 - Chart设置选项
weight: 2
---

## 常见选项

| 选项 | 默认值| 描述 |
| --- | --- | --- |
| `hostname` | " " | `string` - Rancher Server的完全限定域名 |
| `ingress.tls.source` | `rancher` | `string` - 从哪里获得证书 - "rancher, letsEncrypt, secret" |
| `letsEncrypt.email` | " " | `string` - 邮件地址 |
| `letsEncrypt.environment` | `production` | `string` - 有效选项: "staging, production" |
| `privateCA` | false | `bool` - 如果您的证书是自签名CA证书，则设置为true |

<br/>

## 高级选项

| 选项 | 默认值   | 描述 |
| --- | --- | --- |
| `additionalTrustedCAs` | false | `bool` - 查看[额外的授信CA证书](#额外的授信ca证书) |
| `auditLog.destination` | "sidecar" | `string` - Stream to sidecar container console or hostPath volume - "sidecar, hostPath" |
| `auditLog.hostPath` | "/var/log/rancher/audit" | `string` - 主机上的目标日志文件|
| `auditLog.level` | 0 | `int` - - 设置[API审核日志]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/api-auditing/)等级. 0是关闭。 [0-3] |
| `auditLog.maxAge` | 1 | `int` - 保留旧审核日志文件的最大天数 |
| `auditLog.maxBackups` | 1 | `int` - 要保留的最大审计日志文件数 |
| `auditLog.maxSize` | 100 | `int` - 审计日志文件轮换前的最大大小（单位兆） |
| `debug` | false | `bool` - 开启rancher server debug模式 |
| `imagePullSecrets` | [] | `list` - 包含私有镜像仓库登录凭据的Secret资源的名称列表 |
| `proxy` | "" | `string` - string - Rancher的HTTP[S]代理服务器|
| `noProxy` | "127.0.0.0/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16" | `string` - 逗号分隔的主机名列表或不使用代理的IP地址 |
| `resources` | {} | `map` - rancher pod 资源请求和限制 |
| `rancherImage` | "rancher/rancher" | `string` - rancher镜像名称 |
| `rancherImageTag` | same as chart version | `string` - rancher/rancher镜像版本 |
| `tls` | "ingress" | `string` - 有关详细信息，请参阅[外部TLS终止](#外部TLS终止). - "ingress, external" |

<br/>

### API审计日志

启用[API审核日志]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/api-auditing/).

可以在Rancher 集群的`system`项目中启用[rancher日志收集]({{< baseurl >}}/rancher/v2.x/cn/tools/logging/)，就像收集容器日志一样收集此日志

```plain
--set auditLog.level=1
```

默认情况下，启用审核日志记录将在Rancher pod中创建一个sidecar容器。此容器（`rancher-audit-log`）将日志传输到`stdout`。 你可以通过查看Pod日志一下查看审计日志，可以通过开启项目的[日志收集工具]({{< baseurl >}}/rancher/v2.x/cn/tools/logging/)把日志收集到其他地方。

### HTTP Proxy

Rancher需要互联网访问某些功能(helm charts)，如果你的环境需要代理上网，则需要配置 `proxy server`。有些不需要走代理的地址，需要将它们添加到`noProxy`列表中。

```plain
--set proxy="http://<username>:<password>@<proxy_url>:<proxy_port>/"
--set noProxy="127.0.0.0/8\,10.0.0.0/8\,172.16.0.0/12\,192.168.0.0/16"
```

## 额外的授信CA证书

如果您有私有镜像仓库、应用商店或proxy代理，您可能需要向Rancher添加额外的授信CA证书

```plain
--set additionalTrustedCAs=true
```

部署Rancher后，将您的CA证书以pem格式复制到名为`ca-additional.pem`的文件中，并用`kubectl`在命名空间`cattle-system`中创建`tls-ca-additional`密文。

```plain
kubectl -n cattle-system create secret generic tls-ca-additional --from-file=ca-additional.pem
```

### 私有仓库和离线安装

有关使用私有镜像仓库安装Rancher的详细信息，查看[Rancher离线安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/air-gap-installation/)

### 外部TLS终止

我们建议将负载均衡器配置为第4层均衡器，将普通`80/tcp和443/tcp`转发到`Rancher Management`集群节点。集群上的`Ingress Controller`将端口80上的HTTP流量重定向到端口443上的https。

您可以在Rancher集群（ingress）外部的L7负载均衡器上终止SSL/TLS。设置`--set tls=external`选项并将负载均衡器指向所有Rancher集群节点上的`http 80`端口。这将在`http80`端口上公开Rancher入口。

>[Nginx 七层代理配置参考](../../../rke-ha-install/https-l7/nginx/)
>
>**请注意** 允许直接连接到Rancher集群的客户端不会被加密。如果您选择这样做，我们建议您将网络访问权限限制为仅能访问负载均衡器。
>
> 如果您使用的是私有CA签名证书，配置`--set privateCA=true` 并[添加TLS密文](../tls-secrets/#二-私有ca签名证书-可选)

您的负载均衡器必须支持长期的websocket连接，并且需要插入代理头，以便Rancher可以正确地路由链接。

#### Headers要求

* `Host`
* `X-Forwarded-Proto`
* `X-Forwarded-Port`
* `X-Forwarded-For`

#### 推荐超时

* Read Timeout: `1800 seconds`
* Write Timeout: `1800 seconds`
* Connect Timeout: `30 seconds`

#### 健康检查

Rancher将响应端点`/healthz`上的运行状况码`200`进行检查。