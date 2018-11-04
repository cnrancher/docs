---
title: 3 - Helm安装Rancher
weight: 3
---

>**注意：** 对于没有Internet访问的系统，请参考[离线安装Rancher]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/air-gap-installation/)。

## 一、添加Chart仓库地址

使用`helm repo add`命令添加Rancher chart仓库地址,访问[Rancher tag和Chart版本]({{< baseurl >}}/rancher/v2.x/cn/installation/server-tags/)

替换`<CHART_REPO>`为您要使用的Helm仓库分支(即latest或stable）。

```bash
helm repo add rancher-<CHART_REPO> https://releases.rancher.com/server-charts/<CHART_REPO>
```

## 二、安装证书管理器(可选)

>**注意:** 只有Rancher自动生成的证书和LetsEncrypt颁发的证书才需要`cert-manager`。如果是你自己的证书，可使用`ingress.tls.source=secret`参数指定证书，并跳过此步骤。

Rancher依靠`Kubernetes Helm stable`仓库中的[cert-manager](https://github.com/kubernetes/charts/tree/master/stable/cert-manager)来颁发自签名或LetsEncrypt证书.

从Helm stable目录安装`cert-manager`。

```bash
helm install stable/cert-manager \
  --name cert-manager \
  --namespace kube-system
```

## 三、选择SSL配置方式并安装Rancher server

Rancher server设计默认需要开启SSL/TLS配置来保证安全。

证书来源有三种选择：

- `rancher` - 使用Rancher自动生成`CA/Certificates`证书(默认)。
- `letsEncrypt` - 使用`LetsEncrypt`颁发证书。
- `secret` - 使用`Kubernetes Secret`证书配置文件。

### 1、Rancher自动生成证书(默认)

默认情况下，Rancher会自动生成`CA根证书`并使用`cert-manager`颁发证书以访问Rancher server界面。

唯一的要求是将`hostname`配置为访问Rancher的域名地址，使用这种SSL证书配置方式需提前安装[证书管理器](#二-安装证书管理器-可选)。

>修改`hostname`

```bash
helm install rancher-stable/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org
```

### 2、LetsEncrypt

使用[LetsEncrypt](https://letsencrypt.org/)的免费服务发布可信的SSL证书。此配置需要使用http验证，因此给Rancher配置的访问域名地址必须是能够被`互联网访问`。使用这种SSL证书配置方式需提前安装[证书管理器](#二-安装证书管理器-可选)。

>修改`hostname`、`LetsEncrypt`

```bash
helm install rancher-stable/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=letsEncrypt \
  --set letsEncrypt.email=me@example.org
```

### 3、Kubernetes Secret证书

通过你自己的证书创建`Kubernetes Secrets`，以供Rancher使用。

>修改`hostname`

```bash
helm install rancher-stable/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=secret
```

接着根据[添加TLS密文](./tls-secrets)创建证书密文，以便`Rancher和Ingress Controller`可以使用。

>**注意:**
>1.证书对应的`域名`需要与`hostname`选项匹配，否则`ingress`将无法代理访问Rancher。\
>2.如果你使用的是私有CA签名证书，需要添加`--set privateCA=true`

### 4、高级配置

Rancher chart有许多配置选项,可用于自定义安装以适合你的特定环境。以下是一些常见的高级方案:

- [HTTP代理配置](./chart-options/#http-proxy)
- [私有镜像仓库](./chart-options/#private-registry)
- [外部负载均衡器TLS](./chart-options/#external-tls-termination)

有关设置选项的完整列表，请查看[Chart Options](./chart-options/)。

### 5、保存配置参数

确保保存了所有的配置参数，Rancher下一次升级时，helm需要使用相同的配置参数来运行新版本Rancher。

## 四、故障排除

[故障排除]({{< baseurl >}}/rancher/v2.x/cn/faq/troubleshooting-helm/)
