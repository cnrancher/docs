---
title:  6 - Pipeline(CI/CD)
weight: 6
---
>**注意:**
>
>- 对于Rancher v2.1来说，CI/CD有新的改进。因此，如果在使用v2.0.x时配置了CI/CD，则必须在升级到v2.1后重新配置它们。
>- 仍在使用v2.0.x？请参阅[以前的版本]({{< baseurl >}}/rancher/v2.x/cn/tools/pipelines/docs-for-v2.0.x).

`Pipeline`把软件交付过程分成几个不同的阶段，使开发人员能够尽可能快速、高效地提供新的软件。在Rancher中，您可以为每个Rancher项目配置`Pipeline`。

`Pipeline`阶段:

- **克隆:**

    每次将代码推送到存储库时，`Pipeline`都会自动克隆存储库到本地。

- **编译:**

    代码克隆完成后，根据编译参数自动进行代码编译。在编译过程中，通常会通过自动化工具来对代码的进行测试。

- **构建:**

    编译完成并生成执行文件后，利用`Dockerfile`文件自动构建镜像。

- **推送:**

    镜像构建完成后，它会自动发布到Docker镜像仓库。

- **部署:**

    对于在测试阶段的软件，在镜像推送镜像仓库后，可通过自动部署把应用部署到测试环境。

## 概述

Rancher Pipeline提供简单的CI/CD体验。使用它自动签出代码、运行构建、执行测试、发布docker镜像，以及将应用部署到Kubernetes集群。

您可以在Rancher中为每个项目配置Pipeline，每个项目都可以有单独的配置和设置。

用户可以通过以下任一方式读取和编辑管道配置：

- 使用Rancher UI。
- 使用Git CLI等工具更新存储库中的配置，以使用最新的CI定义触发构建。

>**注意:** Rancher Pipeline提供简单的CI/CD体验，但它不能提供全面的功能和灵活性，也不能替代您的团队使用的企业级Jenkins或其他CI工具。

## 支持的代码管理平台

Rancher pipelines目前支持GitHub和GitLab(从Rancher v2.1.0开始提供)。

## Pipelines如何工作

在某个项目中配置pipeline时，将自动创建专门用于pipeline的命名空间并部署以下组件：

- **Jenkins:**

    Pipeline的构建引擎。由于项目用户不直接与Jenkins交互，因此它被管理和锁定。

    >**注意:**  目前暂不支持对接外部Jenkins。

- **Docker Registry:**

    Out-of-the-box, the default target for your build-publish step is an internal Docker Registry. However, you can make configurations to push to a remote registry instead. The internal Docker Registry is only accessible from cluster nodes and cannot be directly accessed by users. Images are not persisted beyond the lifetime of the pipeline and should only be used in pipeline runs. If you need to access your images outside of pipeline runs, please push to an external registry.

- **Minio:**

    Minio存储用于存储Pipeline执行的日志。

  >**Note:** The managed Jenkins instance works statelessly, so don't worry about its data persistency. The Docker Registry and Minio instances use ephemeral volumes by default, which is fine for most use cases. If you want to make sure pipeline logs can survive node failures, you can configure persistent volumes for them, as described in [data persistency for pipeline components]({{< baseurl >}}/rancher/v2.x/en/tools/pipelines/configurations/#data-persistency-for-pipeline-components).

## Pipeline 触发

After you configure a pipeline, you can trigger it using different methods:

- **手动:**

    After you configure a pipeline, you can trigger a build using the latest CI definition from either Rancher UI or Git CLI.  When a pipeline execution is triggered, Rancher dynamically provisions a Kubernetes pod to run your CI tasks and then remove it upon completion.

- **自动:**

    When you enable a repository for a pipeline, webhooks are automatically added to the version control system. When project users interact with the repo—push code, open pull requests, or create a tag—the version control system sends a webhook to Rancher Server, triggering a pipeline execution.

    To use this automation, webhook management permission is required for the repo. Therefore, when users authenticate and fetch their repositories, only those on which they have admin permission will be shown.
