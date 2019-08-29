---
title: 4 - Pipeline 变量参数
weight: 4
---

为方便起见，下面的变量可用于管道配置脚本。在管道执行期间，这些变量被元数据替换。您可以以${VAR_NAME}的形式引用它们。

变量名           | 描述
------------------------|------------------------------------------------------------
`CICD_GIT_REPO_NAME`      | repo名称 (省略Github组织).
`CICD_GIT_URL`            | Git存储库的URL。
`CICD_GIT_COMMIT`         | 正在执行的Git commit ID。
`CICD_GIT_BRANCH`         | 这个事件的Git分支。
`CICD_GIT_REF`            | Git reference specification of this event.
`CICD_GIT_TAG`            | Git tag name, set on tag event.
`CICD_EVENT`              | Event that triggered the build (`push`, `pull_request` or `tag`).
`CICD_PIPELINE_ID`        | Rancher ID for the pipeline.
`CICD_EXECUTION_SEQUENCE` | Build number of the pipeline.
`CICD_EXECUTION_ID`       | Combination of `{CICD_PIPELINE_ID}-{CICD_EXECUTION_SEQUENCE}`.
`CICD_REGISTRY`           | 上一个发布镜像步骤的Docker镜像库的地址，可以在`Deploy YAML`步骤的Kubernetes清单文件中找到。
`CICD_IMAGE`              | 上一个发布镜像步骤构建的镜像名称，可以在`Deploy YAML`的Kubernetes清单文件中找到。它不包含镜像标签。<br/><br/> [例如](https://github.com/rancher/pipeline-example-go/blob/master/deployment.yml)

## 完整`.rancher-pipeline.yml`示例

```yaml
# example
stages:
  - name: Build something
    # Conditions for stages
    when:
      branch: master
      event: [ push, pull_request ]
    # Multiple steps run concurrently
    steps:
    - runScriptConfig:
        image: busybox
        shellScript: echo ${FIRST_KEY} && echo ${ALIAS_ENV}
      # Set environment variables in container for the step
      env:
        FIRST_KEY: VALUE
        SECOND_KEY: VALUE2
      # Set environment variables from project secrets
      envFrom:
      - sourceName: my-secret
        sourceKey: secret-key
        targetKey: ALIAS_ENV
    - runScriptConfig:
        image: busybox
        shellScript: date -R
      # Conditions for steps
      when:
        branch: [ master, dev ]
        event: push
  - name: Publish my image
    steps:
    - publishImageConfig:
        dockerfilePath: ./Dockerfile
        buildContext: .
        tag: rancher/rancher:v2.0.0
        # Optionally push to remote registry
        pushRemote: true
        registry: reg.example.com
  - name: Deploy some workloads
    steps:
    - applyYamlConfig:
        path: ./deployment.yml
# branch conditions for the pipeline
branch:
  include: [ master, feature/*]
  exlclude: [ dev ]

```
