---
title: 1 - Ingress Controllers
weight: 1
---

默认情况下，RKE在所有可调度节点上部署NGINX Ingress Controllers。

> **注意:** 从v0.1.8开始，只有worker被认为是可调度节点，但在v0.1.8之前，worker和controlplane节点被认为是可调度节点。

RKE将ingress控制器部署为`hostnetwork: true`网络模式运行的`Daemon Set`应用，因此在部署控制器的每个节点上都将使用`80`和`443`端口。

## 调度 Ingress Controllers

如果只想在特定节点上部署ingress控制器，可以为ingress设置一个`node_selector`，`node_selector`中的标签需要与要部署的ingress控制器的节点上的标签匹配。

```yaml
nodes:
    - address: 1.1.1.1
      role: [controlplane,worker,etcd]
      user: root
      labels:
        app: ingress

ingress:
    provider: nginx
    node_selector:
      app: ingress
```

## 禁用默认Ingress Controller

您可以通过在集群配置中为`ingress.provider设置为none`来禁用默认控制器。

```yaml
ingress:
    provider: none
```

## 配置 NGINX Ingress Controller

对于NGINX的配置，Kubernetes中有可用的配置选项。有一个[NGINX配置映射的选项列表](https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/configmap.md)、[命令行extra_args](https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/cli-arguments.md)和[annotation](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)。

```yaml
ingress:
    provider: nginx
    options:
      map-hash-bucket-size: "128"
      ssl-protocols: SSLv2
    extra_args:
      enable-ssl-passthrough: ""
```

## 配置NGINX默认证书

在配置具有TLS终止的ingress对象时，必须为其提供用于`加密/解密`的证书。您可以设置一个默认使用的自定义证书，而不需要每次配置ingress规则时候都设置ssl证书。在使用通配符证书的环境中，设置默认证书特别有用，因为证书可以应用于多个子域。

1. 以PEM编码的形式获取或生成证书密钥对。

2. 使用以下命令通过您的PEM编码的证书生成一个Kubernetes secret，注意替换命令中的证书名称。

    ```bash
    kubectl --kubeconfig=kube_configxxx.yml -n   ingress-nginx create secret tls ingress-default-cert --cert=mycert.cert --key=mycert.key -o yaml --dry-run=true > ingress-default-cert.yaml
    ```

3. 把`ingress-default-cert.yml`的内容复制到您的RKE`cluster.yml`文件中。例如:

    ```yaml
    addons: |-
      ---
      apiVersion: v1
      data:
        tls.crt: [ENCODED CERT]
        tls.key: [ENCODED KEY]
      kind: Secret
      metadata:
        creationTimestamp: null
        name: ingress-default-cert
        namespace: ingress-nginx
      type: kubernetes.io/tls
    ```

4. 使用下面的`default-ssl-certificate`参数定义您的ingress资源，该参数引用了我们之前在您的 `cluster.yml`中`extra_args`下创建的秘密:

    ```yaml
    ingress:
      provider: "nginx"
      extra_args:
        default-ssl-certificate: "ingress-nginx/ingress-default-cert"
    ```

5. **可选:** 如果您想将默认证书应用于集群中已经存在的ingress，则必须删除NGINX ingress控制器pods，以便`extra_args`应用到pods。

    ```bash
    kubectl --kubeconfig=kube_configxxx.yml delete  pod -l app=ingress-nginx -n ingress-nginx
    ```
