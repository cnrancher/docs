---
title: 3 - 私有镜像仓库
weight: 3
---

RKE支持配置多个私有Docker镜像仓库的功能，通过传入私有Docker镜像仓库和凭据，允许节点从这些私有Docker镜像仓库中拉取镜像。

```yaml
private_registries:
    - url: registry.com
      user: Username
      password: password
    - url: myregistry.com
      user: myuser
      password: mypassword
```

> **注意:** 如果使用的是Docker Hub镜像仓库，则可以省略url或设置为`docker.io`。

## 默认镜像仓库

从`v0.1.10`开始，RKE支持为系统镜像指定默认私有仓库地址。假设设置`registry.com`为所有系统镜像的默认镜像仓库地址，rke将转换`rancher/rke-tools:v0.1.14`成`registry.com/rancher/rke-tools:v0.1.14`。

```yaml
private_registries:
    - url: registry.com
      user: Username
      password: password
      is_default: true # 所有系统镜像都将从此镜像仓库中拉取。
```

## 离线设置

默认情况下，所有系统镜像都从DockerHub中拉取。如果您所在的环境无法访问互联网，则需要创建一个包含所有系统镜像的私有镜像仓库。

从v0.1.10开始，可以指定默认镜像仓库，以便所有系统镜像都从此镜像仓库拉取。您可以使用`rke config --system-images`命令获取默认系统镜像列表。
