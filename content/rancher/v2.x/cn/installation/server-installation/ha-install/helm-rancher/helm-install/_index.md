---
title: 3 - 安装配置Helm
weight: 3
---

>**注意** helm使用需要kubectl，点击了解[安装和配置kubectl]({{< baseurl >}}/rancher/v2.x/cn/installation/kubectl/)。

Helm是Kubernetes首选的包管理工具。Helm`charts`为Kubernetes YAML清单文档提供模板语法。使用Helm，我们可以创建可配置的部署，而不仅仅是使用静态文件。有关创建自己的`charts`的更多信息，请查看[https://helm.sh/](https://helm.sh/)文档。Helm有两个部分：Helm客户端(helm)和Helm服务端(Tiller)。

## 一、配置Helm客户端访问权限

Helm在集群上安装`tiller`服务以管理`charts`. 由于RKE默认启用RBAC, 因此我们需要使用`kubectl`来创建一个`serviceaccount`，`clusterrolebinding`才能让`tiller`具有部署到集群的权限。

- 在kube-system命名空间中创建`ServiceAccount`；
- 创建`ClusterRoleBinding`以授予tiller帐户对集群的访问权限
- `helm`初始化`tiller`服务

    ```bash
    kubectl -n kube-system create serviceaccount tiller
    kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
    ```

## 二、安装Helm客户端

Helm 客户端可以从源代码安装，也可以从预构建的二进制版本安装。

### 1、从二进制版本

1. 下载你想要的版本[releases](https://github.com/kubernetes/helm/releases)

2. 解压缩(tar -zxvf helm-v2.x.x-linux-amd64.tgz)

3. `helm`在解压后的目录中找到二进制文件，并将其移动到所需的位置(mv linux-amd64/helm /usr/local/bin/helm)

    到这里，你应该可以运行客户端了：`helm help`。

### 2、通过 Homebrew(macOS)安装

Kubernetes社区的成员为Homebrew贡献了Helm,这个通常是最新的。

```bash
brew install kubernetes-helm
```

(注意：emacs-helm 也是一个软件，这是一个不同的项目。)

### 3、从Chocolatey(Windows)

Kubernetes社区成员为Chocolatey贡献了Helm包,这个软件包通常是最新的。

```bash
choco install kubernetes-helm
```

### 4、从脚本

Helm现在有一个安装shell脚本，将自动获取最新版本的`Helm`客户端并在本地安装。可以获取该脚本，然后在本地执行它。这种方法也有文档指导，以便可以在运行之前仔细阅读并理解它在做什么。

```bash
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
  chmod 700 get_helm.sh
  ./get_helm.sh
```

```bash
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
```

也可以做到这一点。

### 5、从金丝雀 (Canary) 构建

`Canary`版本是从最新的主分支构建的Helm软件版本。它们不是正式版本，可能不稳定。但是，他们提供了测试最新功能的机会。\
`Canary`版本Helm二进制文件存储在Kubernetes Helm GCS存储中。以下是常见构建的链接：

- [Linux AMD64](https://kubernetes-helm.storage.googleapis.com/helm-canary-linux-amd64.tar.gz)
- [macOS AMD64](https://kubernetes-helm.storage.googleapis.com/helm-canary-darwin-amd64.tar.gz)
- [Experimental Windows AMD64](https://kubernetes-helm.storage.googleapis.com/helm-canary-windows-amd64.zip)

### 6、源代码方式(Linux，macOS)

从源代码构建Helm的工作稍微多一些，但如果你想测试最新的(预发布)Helm版本，那么这是最好的方法。\
你必须有一个安装`Go`工作环境 。

```bash
cd $GOPATH
  mkdir -p src/k8s.io
  cd src/k8s.io
  git clone https://github.com/kubernetes/helm.git
  cd helm
  make bootstrap build
```

该`build`目标编译`helm`并将其放置在`bin/helm`目录。Tiller也会编译，并且被放置在`bin/tiller`目录。

## 三、安装Helm Server(Tiller)

Helm的服务器端部分Tiller,通常运行在Kubernetes集群内部。但是对于开发，它也可以在本地运行，并配置为与远程Kubernetes集群通信。

### 1、快捷集群内安装

安装`tiller`到集群中最简单的方法就是运行`helm init`。这将验证`helm`本地环境设置是否正确(并在必要时进行设置)。然后它会连接到`kubectl`默认连接的K8S集群(`kubectl config view`)。一旦连接，它将安装`tiller`到`kube-system`命名空间中。

`helm init`自定义参数:

- `--canary-image` 参数安装金丝雀版本;
- `--tiller-image` 安装特定的镜像(版本);
- `--kube-context` 使用安装到特定集群;
- `--tiller-namespace` 用一个特定的命名空间(namespace)安装;

>**注意:** 1、RKE默认启用RBAC,所以在安装`tiller`时需要指定`ServiceAccount`。\
2、`helm init`在缺省配置下，会去谷歌镜像仓库拉取`gcr.io/kubernetes-helm/tiller`镜像，在Kubernetes集群上安装配置Tiller；由于在国内可能无法访问`gcr.io、storage.googleapis.com`等域名，可以通过`--tiller-image`指定私有镜像仓库镜像。点击查询[tiller镜像版本](https://hub.docker.com/r/hongxiaolu/tiller/tags/)。 \
3、`helm init`在缺省配置下，会利用`https://kubernetes-charts.storage.googleapis.com`作为缺省的`stable repository`地址,并去更新相关索引文件。在国内可能无法访问`storage.googleapis.com`地址, 可以通过`--stable-repo-url`指定`chart`国内加速镜像地址。 \
4、如果你是离线安装`Tiller`, 假如没有内部的`chart`仓库, 可通过添加`--skip-refresh`参数禁止`Tiller`更新索引。

执行以下命令在Rancher中安装Tiller：

```bash
helm init --service-account tiller   --tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.11.0 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```

`helm init`以后，可以运行`kubectl get pods --namespace kube-system`并看到Tiller正在运行。

一旦安装了Tiller，运行helm version会显示客户端和服务器版本。(如果它仅显示客户端版本， helm则无法连接到服务器, 使用`kubectl`查看是否有任何tiller Pod 正在运行。)

除非设置`--tiller-namespace`或`TILLER_NAMESPACE`参数，否则Helm将在命名空间`kube-system`中查找Tiller。

### 2、安装Tiller金丝雀版本

Canary镜像是从master分支建立的。他们可能不稳定，但他们提供测试最新功能的机会。安装`Canary`镜像最简单的方法是`helm init与--canary-image`参数一起使用：

```bash
helm init --service-account tiller --canary-image
```

这将使用最近构建的容器镜像。可以随时使用`kubectl`删除`kube-system`命名空间中的`Tiller deployment`来卸载Tiller。

### 3、本地运行Tiller

对于开发而言，有时在本地运行Tiller更容易，将其配置为连接到远程Kubernetes集群。上面介绍了构建部署 Tiller的过程。一旦tiller构建部署完成，只需启动它：

```bash
bin/tiller
Tiller running on:44134
```

当Tiller在本地运行时，它将尝试连接到由`kubectl`配置的Kubernetes集群。(运行kubectl config view以查看是哪个集群。)

必须告知helm连接到这个新的本地Tiller主机，而不是连接到集群中的一个。有两种方法可以做到这一点。第一种是在命令行上指定`--host`选项。第二个是设置`$HELM_HOST`环境变量。

```bash
  export HELM_HOST=localhost:44134
  helm version # Should connect to localhost.
  Client: &version.Version{SemVer:"v2.0.0-alpha.4", GitCommit:"db...", GitTreeState:"dirty"}
  Server: &version.Version{SemVer:"v2.0.0-alpha.4", GitCommit:"a5...", GitTreeState:"dirty"}
```

>注意，即使在本地运行，Tiller也会将安装的release配置存储在Kubernetes内的ConfigMaps中。

## 四、升级Tiller

从Helm 2.2.0开始，Tiller可以升级使用`helm init --upgrade`。对于旧版本的Helm或手动升级，可以使用`kubectl`修改Tiller容器镜像

> 点击查询[新版Tiller镜像](https://hub.docker.com/r/hongxiaolu/tiller/tags/)

```bash
  export TILLER_TAG=<new_tag> ;
  kubectl --namespace=kube-system set image deployments/tiller-deploy tiller=gcr.io/kubernetes-helm/tiller:$TILLER_TAG
```

将显示以下信息：

```bash
deployment "tiller-deploy" image updated
```

设置`TILLER_TAG=canary`将获得master版本的最新快照。

## 五、删除或重新安装Tiller

由于Tiller将其数据存储在Kubernetes ConfigMaps中，因此可以安全地删除并重新安装Tiller，而无需担心丢失任何数据。推荐删除Tiller的方法是使用`kubectl delete deployment tiller-deploy --namespace kube-system`或更简洁使用`helm reset`。

然后可以从客户端重新安装Tiller：

```bash
helm init
```

## 六、高级用法

helm init提供了额外的参数，用于在安装之前修改Tiller的deployment manifest。

### 1、使用`--node-selectors`

`--node-selectors`参数允许我们指定调度Tiller Pod所需的节点标签。

下面的例子将在nodeSelector属性下创建指定的标签。

```bash
helm init --node-selectors "beta.kubernetes.io/os"="linux"
```

已安装的deployment manifest将包含我们的节点选择器标签。

```yaml
...
spec:
    template:
      spec:
        nodeSelector:
          beta.kubernetes.io/os: linux
...
```

### 2、使用--override

`--override`允许指定Tiller的deployment manifest的属性。与在Helm其他地方`--set`使用的命令不同，`helm init --override`修改最终`manifest`的指定属性(没有 "values" 文件)。因此，可以为deployment manifest中的任何有效属性指定任何有效值。

### 3、覆盖注释

在下面的示例中，我们使用`--override`添加修订版本属性并将其值设置为`1`。

```bash
helm init --override metadata.annotations."deployment\.kubernetes\.io/revision"="1"
```

输出：

```yaml
apiVersion: extensions/v1beta1
  kind: Deployment
  metadata:
    annotations:
      deployment.kubernetes.io/revision: "1"
  ...
```

### 4、覆盖亲和性

在下面的例子中，我们为节点设置了亲和性属性。`--override`可以组合来修改同一列表项的不同属性。

```bash
helm init --override "spec.template.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution[0].weight"="1" --override "spec.template.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution[0].preference.matchExpressions[0].key"="e2e-az-name"
```

指定的属性组合到`preferredDuringSchedulingIgnoredDuringExecution`属性的第一个列表项中。

```yaml
...
spec:
    strategy: {}
    template:
      ...
      spec:
        affinity:
          nodeAffinity:
            preferredDuringSchedulingIgnoredDuringExecution:
            - preference:
                matchExpressions:
                - key: e2e-az-name
                  operator: ""
              weight: 1
...
```

### 5、使用--output

`--output`参数允许我们跳过安装Tiller的deployment manifest，并以JSON或YAML格式简单地将 deployment manifest输出到标准输出stdout。然后可以使用`jq`类似工具修改输出，并使用`kubectl`手动安装。

在下面的例子中，我们`helm init`用`--output json`参数执行。

```bash
helm init --output json
```

Tiller安装被跳过，manifest以JSON格式输出到stdout。

```plain
"apiVersion": "extensions/v1beta1",
  "kind": "Deployment",
  "metadata": {
      "creationTimestamp": null,
      "labels": {
          "app": "helm",
          "name": "tiller"
      },
      "name": "tiller-deploy",
      "namespace": "kube-system"
  },
  ...
```

### 6、存储后端

默认情况下，tiller将安装release信息存储在其运行的命名空间中的ConfigMaps中。从Helm2.7.0开始，现在有一个Secrets用于存储安装release信息的beta存储后端。添加了这个功能是为和Kubernetes的加密`Secret`一起，保护chart的安全性。\
要启用secrets后端，需要使用以下选项启动Tiller:

```bash
helm init --override 'spec.template.spec.containers[0].command'='{/tiller,--storage=secret}'
```

目前，如果想从默认后端切换到secrets后端，必须自行为此进行迁移配置信息。当这个后端从beta版本毕业时，将会有更正式的移徙方法。

## [下一步: Helm安装Rancher](../rancher-install/)
