---
title: 1 - Pipeline 术语
weight: 1
---

在设置Pipeline时，了解一些相关术语会很有帮助。

- **Pipeline:**

    Pipeline由阶段和步骤组成。它定义了构建、测试和部署代码的过程。Rancher Pipeline使用[Pipeline作为代码模型](https://jenkins.io/doc/book/pipeline-as-code/) - Pipeline配置文件存放在源代码存储库中，使用文件名`.rancher-pipeline.yml或.rancher-pipeline.yml`。

- **Stages:**

    Pipeline阶段包含多个步骤。`阶段`按Pipeline文件中定义的顺序执行，而`阶段中的步骤`同时执行。当前一阶段中的所有步骤完成而没有失败时，下一阶段开始。

- **Steps:**

    Pipeline步骤在指定阶段内执行。如果退出时状态码不为`0`,则步骤将失败。如果步骤退出抛出故障代码，则整个Pipeline将失败并终止。

- **Workspace:**

    工作空间是所有Pipeline步骤共享的工作目录。在Pipeline的开始，源代码被克隆到工作区，每个步骤的命令都在工作区中被引用。在Pipeline执行期间，前一步骤中的生成的文件将在以后的步骤中可用。工作目录是一个临时卷，在Pipeline执行完成后将使用执行程序窗口清除。
