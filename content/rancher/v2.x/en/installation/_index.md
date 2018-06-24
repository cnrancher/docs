---
title: 安装
weight: 4
---
本章节包含在开发和生产环境中安装Rancher的说明

从以下类型中选择：

- [单节点安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/single-node-install)

	在这个简单的安装场景中，您可以在单个Linux主机上安装RANCHER；

- [单个节点安装+外部负载均衡器]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/single-node-install-external-lb/)

	在此场景下，把Rancher安装在单个Linux主机上，并使用外部负载平衡器/代理来访问它；

-  [HA安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-server-install/)

		此安装方案将创建一个专用于Rancher服务以HA配置运行的新Kubernetes集群；

-  [HA安装+外部负载均衡器]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-server-install-external-lb/)

		此安装方案将创建一个专用于Rancher服务以HA配置运行的新Kubernetes集群，并把负载平衡器位于HA配置之前；

-  [离线安装]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/air-gap-installation/)

		在没有Internet连接的环境中安装Rancher服务器；

本节还包括Rancher配置和维护的帮助内容：

-  [备份和恢复]({{< baseurl >}}/rancher/v2.x/cn/installation/backups-and-restoration/)

-  [端口需求]({{< baseurl >}}/rancher/v2.x/cn/installation/references/)

-  [Rancher HTTP代理配置]({{< baseurl >}}/rancher/v2.x/cn/installation/proxy-configuration/)

	如果您的环境需要代理访问互联网，此页面提供有关如何为您的Rancher配置代理；
