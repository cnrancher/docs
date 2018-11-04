---
title: 2 - Chart设置选项
weight: 2
---

## 常见选项

| 选项 | 默认值| 描述 |
| --- | --- | --- |
| `hostname` | " " | `string` - Rancher Server的完全限定域名 |
| `ingress.tls.source` | `rancher` | `string` - Where to get the cert for the ingress. - "rancher, letsEncrypt, secret" |
| `letsEncrypt.email` | " " | `string` - 邮件地址 |
| `letsEncrypt.environment` | `production` | `string` - 有效选项: "staging, production" |
| `privateCA` | false | `bool` - 如果您的证书是自签名CA证书，则设置为true |

<br/>

## 高级选项

| 选项 | 默认值   | 描述 |
| --- | --- | --- |
| `additionalTrustedCAs` | false | `bool` - See [Additional Trusted CAs](#additional-trusted-cas) |
| `auditLog.destination` | "sidecar" | `string` - Stream to sidecar container console or hostPath volume - "sidecar, hostPath" |
| `auditLog.hostPath` | "/var/log/rancher/audit" | `string` - log file destination on host |
| `auditLog.level` | 0 | `int` - set the [API Audit Log]({{< baseurl >}}/rancher/v2.x/en/installation/api-auditing) level. 0 is off. [0-3] |
| `auditLog.maxAge` | 1 | `int` - maximum number of days to retain old audit log files |
| `auditLog.maxBackups` | 1 | `int` - maximum number of audit log files to retain |
| `auditLog.maxSize` | 100 | `int` - maximum size in megabytes of the audit log file before it gets rotated |
| `debug` | false | `bool` - set debug flag on rancher server |
| `imagePullSecrets` | [] | `list` - list of names of Secret resource containing private registry credentials |
| `proxy` | "" | `string` - string - HTTP[S] proxy server for Rancher |
| `noProxy` | "127.0.0.0/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16" | `string` - comma separated list of hostnames or ip address not to use the proxy |
| `resources` | {} | `map` - rancher pod resource requests & limits |
| `rancherImage` | "rancher/rancher" | `string` - rancher image source |
| `rancherImageTag` | same as chart version | `string` - rancher/rancher image tag |
| `tls` | "ingress" | `string` - See [External TLS Termination](#external-tls-termination) for details. - "ingress, external" |

<br/>

### API审计日志

Enabling the [API Audit Log](https://rancher.com/docs/rancher/v2.x/en/installation/api-auditing/).

You can collect this log as you would any container log. Enable the [Logging service under Rancher Tools](https://rancher.com/docs/rancher/v2.x/en/tools/logging/) for the `System` Project on the Rancher server cluster.

```plain
--set auditLog.level=1
```

By default enabling Audit Logging will create a sidecar container in the Rancher pod. This container (`rancher-audit-log`) will stream the log to `stdout`.  You can collect this log as you would any container log. Enable the [Logging service under Rancher Tools](https://rancher.com/docs/rancher/v2.x/en/tools/logging/) for the Rancher server cluster or System Project.

Set the `auditLog.destination` to `hostPath` to forward logs to volume shared with the host system instead of streaming to a sidecar container. When setting the destination to `hostPath` you may want to adjust the other auditLog parameters for log rotation.

### HTTP Proxy

Rancher requires internet access for some functionality (helm charts). Use `proxy` to set your proxy server.

Add your IP exceptions to the `noProxy` list. Make sure you add the Service cluster IP range (default: 10.43.0.1/16) and any worker cluster `controlplane` nodes. Rancher supports CIDR notation ranges in this list.

```plain
--set proxy="http://<username>:<password>@<proxy_url>:<proxy_port>/"
--set noProxy="127.0.0.0/8\,10.0.0.0/8\,172.16.0.0/12\,192.168.0.0/16"
```

## 额外的受信任的ca

If you have private registries, catalogs or a proxy that intercepts certificates, you may need to add additional trusted CAs to Rancher.

```plain
--set additionalTrustedCAs=true
```

Once the Rancher deployment is created, copy your CA certs in pem format into a file named `ca-additional.pem` and use `kubectl` to create the `tls-ca-additional` secret in the `cattle-system` namespace.

```plain
kubectl -n cattle-system create secret generic tls-ca-additional --from-file=ca-additional.pem
```

### 私有仓库和离线安装

See [Installing Rancher - Air Gap]({{< baseurl >}}/rancher/v2.x/en/installation/air-gap-installation/install-rancher/) for details on installing Rancher with a private registry.

### 外部TLS终止

我们建议将负载均衡器配置为第4层均衡器，将普通`80/tcp和443/tcp`转发到`Rancher Management`集群节点。群集上的`Ingress Controller`将端口80上的HTTP流量重定向到端口443上的https。

You may terminate the SSL/TLS on a L7 load balancer external to the Rancher cluster (ingress). Use the `--set tls=external` option and point your load balancer at port http 80 on all of the Rancher cluster nodes. This will expose the Rancher interface on http port 80. Be aware that clients that are allowed to connect directly to the Rancher cluster will not be encrypted. If you choose to do this we recommend that you restrict direct access at the network level to just your load balancer.

> **Note:** If you are using a Private CA signed cert, add `--set privateCA=true` and see [Adding TLS Secrets - Private CA Signed - Additional Steps]({{< baseurl >}}/rancher/v2.x/en/installation/ha/helm-rancher/tls-secrets/#private-ca-signed---additional-steps) to add the CA cert for Rancher.

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

Rancher will respond `200` to health checks on the `/healthz` endpoint.
