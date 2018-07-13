---
title: 安装
weight: 3
---
本节包含在开发和生产环境中安装Rancher的说明

从以下类型中选择：

- [单节点安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/single-node-install)

    在这个简单的安装场景中，你可以在单个Linux主机上安装RANCHER。

- [单个节点安装+外部负载均衡器]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/single-node-install-external-lb/)

    在此场景下，把Rancher安装在单个Linux主机上，并使用外部负载平衡器/代理来访问它。

- [HA安装+外部四层负载(TCP-L4)]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install-external-lb-tcp-l4/)

    此安装方案将创建一个专用于Rancher服务以HA配置运行的新Kubernetes集群，并把外部四层负载(TCP-L4)放置于HA配置之前。

- [HA安装+外部负载均衡器(https-L7)]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install-external-lb-https-l7/)

    此安装方案将创建一个专用于Rancher服务以HA配置运行的新Kubernetes集群，并把外部七层负载(https-L7)放置于HA配置之前。

- [离线安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/air-gap-installation/)

    在没有Internet连接的环境中安装Rancher server。

本节还包括Rancher配置和维护的帮助内容：

- [备份和恢复]({{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/)

- [端口需求]({{< baseurl >}}/rancher/v2.x/cn/installation/references/)

- [Rancher HTTP代理配置]({{< baseurl >}}/rancher/v2.x/cn/installation/proxy-configuration/)

    如果你的环境需要代理访问互联网，此页面提供有关如何为你的Rancher配置代理。

- [生成SSL自签名证书]({{< baseurl >}}/rancher/v2.x/cn/installation/self-signed-ssl/)