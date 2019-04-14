---
title: 4 - Kube配置文件
weight: 4
---

当在安装rancher HA或者UI无法访问时，可以通过`kubectl`命令行工具去访问，参考[kubectl安装]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/kubectl/)。

当RKE部署Kubernetes时，会自动生成kubeconfig配置文件，文件名称为`kube_config_<rke 配置文件名>.yml`。

默认情况下，`kubectl`会检查`~/.kube/config`，可以使用`--kubeconfig`参数指定配置文件，比如:

```bash
kubectl --kubeconfig /custom/path/kube.config get pods
```

通过检查Kubernetes集群的版本来确认kubectl时候正常运行

```bash
kubectl --kubeconfig kube_config_cluster.yml version

Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.0", GitCommit:"fc32d2f3698e36b93322a3465f63a14e9f0eaead", GitTreeState:"clean", BuildDate:"2018-03-27T00:13:02Z", GoVersion:"go1.9.4", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"8+", GitVersion:"v1.8.9-rancher1", GitCommit:"68595e18f25e24125244e9966b1e5468a98c1cd4", GitTreeState:"clean", BuildDate:"2018-03-13T04:37:53Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}
```
