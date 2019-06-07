---
title: 4 - 默认镜像仓库地址
weight: 4
---

1. 登录Rancher并配置默认管理员密码。

1. 进入**Settings**视图。

    ![Settings]({{< baseurl >}}/img/rancher/airgap/settings.png)

1. 查找`system-default-registry`并点击**Edit**。

    ![Edit]({{< baseurl >}}/img/rancher/airgap/edit-system-default-registry.png)

1. 将值改为您的私有仓库地址, e.g. registry.yourdomain.com:port. 不要添加`http:// or https://`前缀

    ![Save]({{< baseurl >}}/img/rancher/airgap/enter-system-default-registry.png)

>**注意:** 如果要在启动rancher/rancher容器时配置``system-default-registry`，可以使用环境变量
>`CATTLE_SYSTEM_DEFAULT_REGISTRY`
>```bash
>#!/bin/sh
>docker run -d -p 80:80 -p 443:443 \
>-v <主机路径>:/var/lib/rancher/ \
>-v /root/var/log/auditlog:/var/log/auditlog \
>-e AUDIT_LEVEL=3 \
>-e CATTLE_SYSTEM_DEFAULT_REGISTRY=<registry.yourdomain.com:port>
><registry.yourdomain.com:port>/>rancher/rancher:v2.0.0
>```
