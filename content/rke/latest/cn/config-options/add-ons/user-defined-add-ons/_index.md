---
title: 5 - 自定义插件
weight: 5
---

除了[network plug- on]({{< baseurl >}}/rke/latest/cn/config-options/add-ons/network-plugins)和[ingress controllers]({{< baseurl >}}/rke/latest/cn/config-options/add-ons/ ings -controllers/)之外，您还可以定义在部署Kubernetes集群之后希望部署的任何附加组件。

可以通过两种方式指定加载项:

- [在线附加组件](#一-在线附加组件)
- [引用加载项的YAML文件](#二-引用加载项的yaml文件)

> **Note:**  当使用自定义附加组件时，`必须`为`所有`资源定义一个名称空间，否则它们将最终部署在`kube-system`命名空间中。

RKE将YAML清单作为configmap上传到Kubernetes集群。然后，它运行一个Kubernetes job，该`job`挂载configmap并使用`kubectl --kubeconfig=kube_configxxx.yml  apply  -f`部署附加组件。

从v0.1.8开始，如果加载项名称相同，则RKE将更新加载项。在v0.1.8之前，通过使用`kubectl --kubeconfig=kube_configxxx.yml edit`更新加载项

## 一、在线附加组件

要在YAML文件中直接定义一个附加组件，请确保使用YAML的块指示器`|-`，因为`addons`指令是一个多行字符串选项。可以使用`——`指令将多个YAML资源定义分隔开来，从而指定它们。

```yaml
addons: |-
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-nginx
      namespace: default
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
```

## 二、引用加载项的YAML文件

使用`addons_include`指令来引用自定义的附加组件的本地yaml文件或URL。

```yaml
addons_include:
    - https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/operator.yaml
    - https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/cluster.yaml
    - /opt/manifests/example.yaml
    - ./nginx.yaml
```
