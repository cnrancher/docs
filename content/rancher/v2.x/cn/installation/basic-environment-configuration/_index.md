---
title: 基础环境配置
weight: 1
---
## 镜像仓库配置

如果是在离线环境下安装kubenetes集群,那么对应的项目需要为公开权限，不用登录即可拉取镜像。因为部署kubenetes属于全局层功能，在全局层下没有镜像仓库功能，无法代理登录私有仓库。

## Dokcer基础配置

1. 配置镜像下载和上传并发数

    从Docker1.12开始，支持自定义下载和上传镜像的并发数，默认值上传为3个并发，下载为5个并发。通过添加"max-concurrent-downloads"和"max-concurrent-uploads"参数对其修改：

    ```json
    "max-concurrent-downloads": 3,
    "max-concurrent-uploads": 5
    ```
2. 配置镜像加速地址

    Docker默认仓库地址是dockerhub，

3. 