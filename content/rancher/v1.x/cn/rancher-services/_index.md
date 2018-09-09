---
title: 基础服务
weight: 500
---



当启动Rancher时，每个[环境]({{< baseurl >}}/rancher/v1.x/cn/configuration/environments/)的创建都基于[环境模版]({{< baseurl >}}/rancher/v1.x/cn/configuration/environments/#什么是环境模版)。在启动一个环境时，你可以在环境模版中选择需要启动的基础设施服务。这些基础设施服务包括编排引擎，[外部DNS]({{< baseurl >}}/rancher/v1.x/cn/infrastructure/cattle/external-dns-service/)，[网络]({{< baseurl >}}/rancher/v1.x/cn/rancher-services/networking/)，[存储]({{< baseurl >}}/rancher/v1.x/cn/rancher-services/storage-service/)，框架服务 (例如: [内部DNS]({{< baseurl >}}/rancher/v1.x/cn/rancher-services/dns-service/)，[Metadata服务]({{< baseurl >}}/rancher/v1.x/cn/rancher-services/metadata-service)，和[健康检查服务]({{< baseurl >}}/rancher/v1.x/cn/infrastructure/cattle/health-checks))。

基础设施服务位于[Rancher应用商店](https://github.com/rancher/rancher-catalog)和[社区应用商店](https://github.com/rancher/community-catalog)中的`infra-templates`文件夹中。Rancher应用商店和社区应用商店是默认开启的，它们提供了一系列可以在环境模版中使用的基础服务。

当创建一个环境模版时，默认开启运行一个环境所需的一系列基础服务。
