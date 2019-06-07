---
title: 1 - 版本选择
weight: 1
---

## 镜像标签

{{< product >}}服务是作为一个Docker镜像分发的，它带有tag标签。标签用于表示镜像中包含的{{< product >}}版本。 如果您需要使用特定标签版本的镜像，需要先拉取该标签版本的镜像。否则，如果本地有该版本的镜像，Docker将优先使用本地镜像。

您可以在 [DockerHub](https://hub.docker.com/r/rancher/rancher/tags/)找到Rancher server镜像:

| Tag                        | Description                                                                                                                                                     |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `rancher/rancher:latest`   | 最新的开发版本，通过我们的CI自动化框架进行构建，该版本不推荐用于生产环境。|
| `rancher/rancher:stable`   | 最新的稳定版本，该版本被推荐用于生产。                             |
| `rancher/rancher:<v2.X.X>` | 可以通过明确指定镜像版本标签来安装特定的Rancher server版本。                                                                         |

>master或-rc或其他后缀的标签都是供{{< product >}}测试团队验证的。不要使用这些标签，这些版本不提供官方的支持。

## Helm Chart类型

类型 | 添加仓库命令 | 仓库描述
-----------|-----|-------------
rancher-latest   | `helm repo add rancher-latest https://releases.rancher.com/server-charts/latest` | Rancher server最新版Helm charts仓库，建议此仓库版本用于测试环境。
rancher-stable  | `helm repo add rancher-stable https://releases.rancher.com/server-charts/stable` | Rancher server稳定版Helm charts仓库，此仓库版本推荐用于生产环境。
<br/>

## Helm Chart版本

运行`helm search rancher`查看Rancher版本

```bash
NAME                      CHART VERSION    APP VERSION    DESCRIPTION
rancher-latest/rancher    2018.10.1            v2.1.0      Install Rancher Server to manage Kubernetes clusters acro...
```

## 切换到不同的Helm Chart仓库

1. 列出当前的Helm chart仓库。

    ```bash
    helm repo list

    NAME          	      URL
    stable        	      https://kubernetes-charts.storage.googleapis.com
    rancher-<CHART_REPO>	https://releases.rancher.com/server-charts/<CHART_REPO>
    ```

2. 删除现有的Helm chart仓库

    ```bash
    helm repo remove rancher-<CHART_REPO>
    ```

3. 添加需要安装的仓库版本，使用(`latest` or `stable`)替换`<CHART_REPO>`。

    ```bash
    helm repo add rancher-<CHART_REPO> https://releases.rancher.com/server-charts/<CHART_REPO>
    ```
