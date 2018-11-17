---
title: 2 - Pipelines 快速入门指南
weight: 2
---

Rancher附带了几个示例存储库，您可以使用它们来熟悉Pipeline。我们建议在使用您自己的存储库配置Pipeline之前，配置和测试与您的环境相似的示例存储库。Rancher包含以下示例存储库：

- Go
- Maven
- php

## 1. 设置代码仓库

默认情况下，禁用示例Pipeline存储库。启用一个（或多个）测试Pipeline功能并查看其工作原理。

1. 进入要运行Pipeline的项目。

1. 从主菜单中，选择Workloads。然后选择`Pipelines`选项卡。

1. 单击**设置代码库**.

    **步骤结果:** 显示示例存储库列表。

    >**注意:** 仅当您尚未获取自己的存储库时，才会显示示例存储库。

1. 选择一个示例仓库，点击**启用** (例如: `https://github.com/rancher/pipeline-example-go.git`),然后点击**完成**。

**结果:**

- 为示例存储库配置了Pipeline，并将其添加到`Pipeline`选项卡。
- 以下工作负载部署到新命名空间：

  - `docker-registry`
  - `jenkins`
  - `minio`

## 2. 运行示例Pipeline

配置示例存储库后，运行管道以查看其工作原理。

1. 从`Pipeline`选项卡中，选择省略号`(...) > 运行`。

  >**注意:** 第一次运行管道时，需要一些时间来拉取相关镜像并运行必要的`Pipeline`组件。要了解示例管道正在执行的操作，请选择`Ellipsis (...) > Edit Config`您的repo。或者，查看`.rancher-pipeline.yml`示例存储库中的文件。

**结果:** 管道运行。您可以在日志中查看结果。

## 下一步？

有关在生产中设置管道的详细信息，请参阅[设置Pipelines]({{< baseurl >}}/rancher/v2.x/cn/tools/pipelines/configurations/).
