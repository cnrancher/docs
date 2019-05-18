---
title: 2 - Rancher
weight: 2
---
## Kubernetes

### 当说Rancher v2.x 建立在Kubernetes上时意味着什么？

Rancher v2.0是一个完整的容器管理平台，基于Kubernetes自定义资源和控制器框架构建。所有功能都编写为CustomResourceDefinition（CRD），它扩展了现有的Kubernetes API，并可以利用RBAC等功能。

### 是否计划使用上游Kubernetes，或继续使用自己的分支?

当您选择让我们创建Kubernetes集群的默认选项时，我们仍将提供我们的发行版，但它将非常接近上游版本。

### Rancher2.x版本是否意味着我们需要重新培训我们在Kubernetes的支持人员？

是的。Rancher将通过`kubectl`使用原生Kubernetes功能，但也将提供我们自己的UI仪表板，允许您部署Kubernetes工作负载，而不必了解Kubernetes的全部复杂性。但是，为了充分利用Kubernetes，我们建议您理解Kubernetes。我们计划通过后续版本改进我们的用户体验，以使Kubernetes更易于使用。

## Cattle

### Rancher v2.0 对Cattle有什么影响?

由于Rancher已经重新设计为基于Kubernetes，因此v2.0不支持Cattle，但是Kubernetes上也有cattle类似的功能。

## 环境和集群

### 我还可以为环境和集群创建模板吗？

不可以. 从2.0开始，环境的概念已经改为Kubernetes集群，并且只支持Kubernetes调度引擎。

### 还可以将现有主机添加到环境中吗?

是。我们仍然提供通过自定义方式创建集群。

## 升级和迁移

### 如何从v1.x迁移到v2.0？

由于将Docker容器转换为Kubernetes pod的技术难度，升级将要求用户将这些工作负载从v1.x迁移到新的v2.0环境中。我们计划在v2.1中增加一个工具，将现有的Rancher Compose文件转换为Kubernetes YAML文件。然后，你将能够在v2.0平台上部署这些工作负载。

### 是否可以从Rancher v1.0升级到v2.0而不会对Cattle和Kubernetes集群造成任何干扰？

目前，我们仍在探索这种情况并收集反馈意见。我们预计您需要启动一个新的Rancher实例，然后在v2.0上重新启动。一旦你转到v2.0，升级就会到位，就像它们在v1.6中一样。

### 可以将OpenShift Kubernetes集群导入v2.0吗？

我们的目标是运行上游Kubernetes集群，因此，Rancher v2.0应该可以与OpenShift一起使用，但我们还没有测试过它。

## 支持

### 那么Rancher v1.6是否计划发布一些长期的支持版本?

这是v1.6的重点。我们将继续改进该版本，修复错误，并至少在2.0发布的未来12个月内继续维护。如有必要，我们将延长维护时间，具体取决于用户迁移到v2.1的速度。

### Rancher v2.0是否支持Docker Swarm和Mesos作为环境类型?

在选择Rancher v2.0中创建环境时，Swarm和Mesos将不再是您可以选择的标准选项。不过，Swarm和Mesos都将继续作为可以部署的Catalog应用程序提供。这是一个艰难的决定，但最终，它还是被采纳了。超过15,000个集群中，只有大约200个在运行Swarm。

### 是否可以使用Rancher v2.0管理Azure容器服务?

是的.

### Windows支持怎么样?

 我们计划基于微软使用Kubernetes和CNI提供的overlay网络新方法，为v2.1提供Windows支持。这个新方法与我们在v2.1中所做的非常匹配，一旦完成，您将能够利用相同的Rancher UX，或者Kubernetes UX。我们正在与微软讨论如何实现这一目标，我们将在今年年底前提供更多信息。
