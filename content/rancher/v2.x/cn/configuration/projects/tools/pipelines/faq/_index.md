---
title:  6 - FAQ
weight: 6
---

## 构建镜像时，提示未验证

![image-20181114160553191](_index.assets/image-20181114160553191-2182753.png)

问题主要是仓库是非`https`地址,导致Docker无法验证登陆镜像仓库。

解决方法:

1. 点击`省略号 > 查看/编辑YAML`

    ![image-20181114162323613](_index.assets/image-20181114162323613-2183803.png)

2. 在`publishImageConfig`下添加环境变量(`env`与`publishImageConfig`同级)，比如:

    ```yaml
        steps:
        - publishImageConfig:
            dockerfilePath: ./Dockerfile
            buildContext: .
            tag: example-helloserver:${CICD_EXECUTION_SEQUENCE}
          env:
            PLUGIN_DEBUG: "true"
            PLUGIN_INSECURE: "true"
    ```

    ![image-20181114162723797](_index.assets/image-20181114162723797-2184043.png)

    [点击了解具体详情](https://github.com/rancher/rancher/issues/16218#issuecomment-432692025)

3. 最后点击提交到代码库