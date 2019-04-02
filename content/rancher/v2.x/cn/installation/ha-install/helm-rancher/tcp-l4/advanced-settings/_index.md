---
title: 5 - Rancher高级设置
weight: 5
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
| `auditLog.destination` | "sidecar" | `string` - 审计日志传输到sidecar container console或者hostPath volume - "sidecar, hostPath" |
| `addLocal` | auto | `string`- 让Rancher检测并导入`local`群集 |
| `auditLog.hostPath` | "/var/log/rancher/audit" | `string` - 主机上的目标日志文件|
| `auditLog.level` | 0 | `int` - - 设置[API审核日志]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/api-auditing/)等级. 0是关闭。 [0-3] |
| `auditLog.maxAge` | 1 | `int` - 保留旧审核日志文件的最大天数 |
| `auditLog.maxBackups` | 1 | `int` - 要保留的最大审计日志文件数 |
| `auditLog.maxSize` | 100 | `int` - 审计日志文件轮换前的最大大小（单位兆） |
| `busyboxImage` | busybox | `string`- 用于收集审计日志的busybox镜像。*注意：自v2.2.0起可用* |
| `debug` | false | `bool` - 开启rancher server debug模式 |
| `extraEnv` | [] | `list`- 为Rancher设置其他环境变量。*注意：从v2.2.0开始可用* |
| `imagePullSecrets` | [] | `list` - 包含私有镜像仓库登录凭据的Secret资源的名称列表 |
| `ingress.extraAnnotations` | {} | `map` - 用于自定义入口的附加注释 |
| `proxy` | "" | `string` - string - Rancher的HTTP[S]代理服务器|
| `noProxy` | "127.0.0.0/8,<br/>10.0.0.0/8,<br/>172.16.0.0/12,<br/>192.168.0.0/16" | `string` - 逗号分隔的主机名列表或不使用代理的IP地址 |
| `resources` | {} | `map` - rancher pod 资源请求和限制 |
| `rancherImage` | "rancher/rancher" | `string` - rancher镜像名称 |
| `rancherImageTag` | same as chart version | `string` - rancher/rancher镜像版本 |
| `tls` | "ingress" | `string` - 有关详细信息，请参阅[外部TLS终止](#外部TLS终止). - "ingress, external" |

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

如果您有私有镜像仓库、应用商店或proxy代理，您可能需要向Rancher添加额外的授信CA证书，[自定义CA根证书]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/custom-ca-root-certificate/)

## TLS设置

在Rancher v2.1.7中，默认TLS配置为仅接受TLS 1.2和安全TLS密码套件,不支持TLS 1.3和TLS 1.3独占密码套件。

参考[TLS设置]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/tls-setting/)

### 私有仓库和离线安装

有关使用私有镜像仓库安装Rancher的详细信息，查看[镜像仓库安装]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/registry/)和[Rancher离线安装]({{< baseurl >}}/rancher/v2.x/cn/installation/air-gap-installation/)
