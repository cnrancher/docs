---
title: 5 - Rancher高级设置
weight: 5
---

## 常见选项

| 选项 | 默认值| 描述 |
| --- | --- | --- |
| `hostname` | " " | `string` - Rancher Server的完全限定域名 |
| `ingress.tls.source` | `rancher` | `string` - 从哪里获得证书 - `rancher, letsEncrypt, secret` |
| `letsEncrypt.email` | " " | `string` - 邮件地址 |
| `letsEncrypt.environment` | `production` | `string` - 选项: `staging, production` |
| `privateCA` | false | `bool` - 如果您的证书是自签名CA证书，则设置为true |

<br/>

## 高级选项

| OPTION                         | DEFAULT VALUE                                                | DESCRIPTION                                                  |
| :----------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `additionalTrustedCAs`         | false                                                        | `bool` - 查看[自定义CA根证书]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/custom-ca-root-certificate/) |
| `addLocal`                     | `auto`                                                       | `string` - 让Rancher检测并导入`local`集群                    |
| `antiAffinity`                 | `preferred`                                                  | `string` -  Rancher pods 反亲和规则 - 可用选项: `preferred, required`  |
| `auditLog.destination`         | `sidecar`                                                    | `string` - 审计日志传输到`sidecar container console`或者`hostPath volume` - 可用选项: `sidecar, hostPath` |
| `auditLog.hostPath`            | `/var/log/rancher/audit`                                     | `string` - 主机上的审计日志文件路径(仅当`auditLog.destination`被设置为`hostPath`时才适用) |
| `auditLog.level`               | 0                                                            | `int` - 设置[API审核日志]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/api-auditing/)等级. 0是关闭。 可用选项: [0-3] |
| `auditLog.maxAge`              | 1                                                            | `int` - 保留旧审计日志文件的最大天数(仅当`auditLog.destination`被设置为`hostPath`时适用) |
| `auditLog.maxBackups`          | 1                                                            | `int` - 要保留的最大审计日志文件数量(仅当`auditLog.destination`被设置为`hostPath`时才适用) |
| `auditLog.maxSize`             | 100                                                          | `int` - 在轮换审计日志文件之前，它的最大大小(以兆为单位)(仅当`auditLog.destination`被设置为`hostPath`时才适用) |
| `busyboxImage`                 | `busybox`                                                    | `string` - 用于收集审计日志的busybox镜像<br/>注意: 从v2.2.0开始可用 |
| `debug`                        | false                                                        | `bool` - set debug flag on rancher server                    |
| `extraEnv`                     | []                                                           | `list` - 为Rancher设置额外的环境变量<br/>*注意:从v2.2.0开始可用* |
| `imagePullSecrets`             | []                                                           | `list` - 包含私有镜像仓库凭据的密文资源的名称列表            |
| `ingress.extraAnnotations`     | {}                                                           | `map` - 用于自定义ingress的附加注释                          |
| `ingress.configurationSnippet` | ””                                                           | `string` - 添加额外的Nginx配置。可用于代理配置。*注: 从v2.0.15、v2.1.10和v2.2.4开始提供* |
| `proxy`                        | ””                                                           | `string` - HTTP[S] proxy server for Rancher                  |
| `noProxy`                      | 127.0.0.0/8,<br />10.0.0.0/8,<br />172.16.0.0/12,<br />192.168.0.0/16 | `string` - 不使用代理的主机名列表或ip地址,逗号分隔.          |
| `resources`                    | {}                                                           | `map` - rancher pod 资源的请求和限制                         |
| `rancherImage`                 | `rancher/rancher`                                            | `string` - rancher镜像名                                     |
| `rancherImageTag`              | same as chart version                                        | `string` - rancher镜像tag                                    |
| `tls`                          | “ingress”                                                    | `string` - 查看[外部七层负载均衡Helm HA部署]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/https-l7/#2-配置ssl并安装rancher-server)了解详细使用. - 可用选项: `ingress, external` |

### API审计日志

启用[API审核日志]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/api-auditing/).

可以在Rancher 集群的`system`项目中启用[rancher日志收集]({{< baseurl >}}/rancher/v2.x/cn/tools/logging/)，就像收集容器日志一样收集此日志

```plain
--set auditLog.level=1
```

默认情况下，启用审核日志记录将在Rancher pod中创建一个sidecar容器。此容器（`rancher-audit-log`）将日志传输到`stdout`。 您可以通过查看Pod日志来查看审计日志，可以通过开启项目的[日志收集工具]({{< baseurl >}}/rancher/v2.x/cn/tools/logging/)把日志收集到其他地方。

### HTTP Proxy

Rancher需要互联网访问某些功能(helm charts)，如果您的环境需要代理上网，则需要配置 `proxy server`。有些不需要走代理的地址，需要将它们添加到`noProxy`列表中。

```plain
--set proxy="http://<username>:<password>@<proxy_url>:<proxy_port>/"
--set noProxy="127.0.0.0/8\,10.0.0.0/8\,172.16.0.0/12\,192.168.0.0/16"
```

## 额外的授信CA证书

如果您有私有镜像仓库、应用商店或proxy代理，您可能需要向Rancher添加额外的授信CA证书，[自定义CA根证书]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/custom-ca-root-certificate/)

## TLS设置

在Rancher v2.1.7中，默认TLS配置为仅接受TLS 1.2和安全TLS密码套件,不支持TLS 1.3和TLS 1.3独占密码套件。

参考[TLS设置]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/tls-setting/)

### 私有仓库和离线安装

有关使用私有镜像仓库安装Rancher的详细信息，查看[镜像仓库安装]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/registry/)和[Rancher离线安装]({{< baseurl >}}/rancher/v2.x/cn/installation/air-gap-installation/)
