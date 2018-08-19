---
title: 2 - 版本标签
weight: 2
---
{{< product >}}服务是作为一个Docker镜像分发的，它带有tag标签。标签用于表示镜像中包含的{{< product >}}版本。 如果你需要使用特定标签版本的镜像，需要先拉取该标签版本的镜像。否则，如果本地有该版本的镜像，Docker将优先使用本地镜像。

你可以在 [DockerHub](https://hub.docker.com/r/rancher/rancher/tags/)找到Rancher 镜像:

- `rancher/rancher:latest`:最新的开发版本，通过我们的CI自动化框架进行构建。该版本不推荐用于生产环境。
- `rancher/rancher:stable`:最新的稳定版本，该版本被推荐用于生产。

master或-rc或其他后缀的标签都是供{{< product >}}测试团队验证的。不要使用这些标签，这些版本不提供官方的支持。