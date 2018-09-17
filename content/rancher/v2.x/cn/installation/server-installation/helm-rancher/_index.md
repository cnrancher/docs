---
title: 4 - Helm安装Rancher
weight: 4
---

Rancher也支持使用Kubernetes的Helm包管理器镜像安装。

{{% accordion id="1" label="一、添加Chart仓库地址" %}}

使用`helm repo add`命令添加Rancher chart仓库地址.

```bash
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
```

{{% /accordion %}}
{{% accordion id="2" label="二、安装证书管理" %}}

>**注意** 只有Rancher自动生成的证书和LetsEncrypt颁发的证书才需要cert-manager。如果是你自己的证书，可使用`ingress.tls.source=secret`参数指定证书，并跳过此步骤。

Rancher依靠`Kubernetes Helm stable`仓库中的[cert-manager](https://github.com/kubernetes/charts/tree/master/stable/cert-manager)来颁发自签名或LetsEncrypt证书.

从Helm stable目录安装`cert-manager`。

```bash
helm install stable/cert-manager \
  --name cert-manager \
  --namespace kube-system
```

{{% /accordion %}}
{{% accordion id="3" label="三、选择SSL配置" %}}

Rancher server设计默认需要开启SSL/TLS配置来保证安全。

证书来源有三种选择：

- rancher - (默认)使用Rancher自动生成CA/Certificates。
- letsEncrypt - 使用LetsEncrypt颁发证书。
- secret - 使用Kubernetes Secret证书配置文件。

### 1、(默认)Rancher自动生成证书

默认情况下，Rancher会自动生成CA并使用cert-manager颁发证书以访问Rancher服务器界面。

唯一的要求是将`hostname`指向访问Rancher的域名地址。

```bash
helm install rancher-stable/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org
```

### 2、LetsEncrypt

使用[LetsEncrypt](https://letsencrypt.org/)的免费服务发布可信的SSL证书。此配置需要使用http验证，因此给Rancher配置的访问域名地址必须是能够在公网访问。

>配置`hostname`、`ingress.tls.source=letEncrypt`和LetsEncrypt设置选项。

```bash
helm install rancher-stable/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=letsEncrypt \
  --set letsEncrypt.email=me@example.org
```

### 3、Kubernetes Secret配置证书

从你自己的证书中创建Kubernetes Secrets以供Rancher使用。

>**注意:** 1.证书的对应的`域名`需要与`hostname`选项匹配，否则ingress将无法代理访问Rancher。\
>2.如果你使用的是私有CA签名证书，需要添加`--set privateCA=true`

配置`hostname`和`ingress.tls.source=secret`

```bash
helm install rancher-stable/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=secret
```

现在Rancher正在运行，请参阅添加[TLS密文](./tls-secrets)以发布证书文件，以便Rancher和Ingress Controller可以使用。

### 4、高级配置

Rancher chart有许多配置选项,可用于自定义安装以适合你的特定环境。以下是一些常见的高级方案:

- [HTTP代理配置](./chart-options/#http-proxy)
- [私有镜像仓库](./chart-options/#private-registry)
- [外部负载均衡器TLS](./chart-options/#external-tls-termination)

有关设置选项的完整列表，请查看[Chart Options](./chart-options/)。

### 5、保存配置参数

确保保存了所有的配置参数，Rancher下一次升级时，helm需要使用相同的配置参数来运行新版本Rancher。

{{% /accordion %}}
{{% accordion id="4" label="四、故障排除" %}}

### 1、[故障排除]({{< baseurl >}}/rancher/v2.x/cn/faq/troubleshooting-helm/)

{{% /accordion %}}
