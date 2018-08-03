---
title: Technical
weight: 1
---

## 如何重置管理员密码？

- 单节点安装

  ```bash
  docker exec -ti <container_id> reset-password
  New password for default admin user (user-xxxxx):
  <new_password>
  ```

- HA安装

  ```bash
  KUBECONFIG=./kube_config_rancher-cluster.yml
  kubectl --kubeconfig $KUBECONFIG exec -n cattle-system $(kubectl   --kubeconfig $KUBECONFIG get pods -n cattle-system -o json | jq -r '.items  [] | select(.spec.containers[].name=="cattle-server") | .metadata.name')   -- reset-password
  New password for default admin user (user-xxxxx):
  <new_password>
  ```

## 怎么样开启debug模式？

### 单节点安装

- Enable

  ```bash
  docker exec -ti <container_id> loglevel --set debug
  OK
  docker logs -f <container_id>
  ```

- 禁用

  ```bash
  docker exec -ti <container_id> loglevel --set info
  OK
  ```

### HA安装

- 启用

  ```bash
  KUBECONFIG=./kube_config_rancher-cluster.yml
  kubectl --kubeconfig $KUBECONFIG exec -n cattle-system $(kubectl   --kubeconfig $KUBECONFIG get pods -n cattle-system -o json | jq -r '.items  [] | select(.spec.containers[].name=="cattle-server") | .metadata.name')   -- loglevel --set debug
  OK
  kubectl --kubeconfig $KUBECONFIG logs -n cattle-system -f $(kubectl   --kubeconfig $KUBECONFIG get pods -n cattle-system -o json | jq -r '.items  [] | select(.spec.containers[].name="cattle-server") | .metadata.name')
  ```

- 禁用

  ```bash
  KUBECONFIG=./kube_config_rancher-cluster.yml
  kubectl --kubeconfig $KUBECONFIG exec -n cattle-system $(kubectl   --kubeconfig $KUBECONFIG get pods -n cattle-system -o json | jq -r '.items  [] | select(.spec.containers[].name=="cattle-server") | .metadata.name')   -- loglevel --set info
  OK
  ```

## ClusterIP无法ping通？

ClusterIP是一个虚拟IP，不会响应ping。测试ClusterIP配置是否正确的最好方法是使用`curl`来访问IP和端口以查看它是否响应。

## 我在哪里可以管理主机模板？

打开你的帐户菜单（右上角），并选择`主机模板`。

## 为什么我的L4层负载均衡服务处于“挂起”状态？

L4层负载均衡器创建为`type：LoadBalancer`，在Kubernetes中，这需要云提供商或控制器能够满足这些请求，否则这些将永远处于“挂起”状态。 了解更多[云提供商]({{< baseurl >}}/rancher/v2.x/cn/concepts/clusters/cloud-providers/) 或者 [Create External Load Balancer](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/)

## Rancher的状态存储在什么地方？

- 单节点安装

  在rancher/rancher容器的内置etcd中，映射与宿主机的`/var/lib/rancher`目录下。

- HA安装

  RKE部署集群指定的ETCD中，默认与Kubernetes共有一套ETCD服务。

## 如何确定支持的Docker版本？

我们遵循经过Kubernetes官方验证过的Docker版本，已验证的Docker版本可以在Kubernetes的[发版记录](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.10.md#external-dependencies)中找到。

## 我如何访问Rancher创建的节点？

可以通过**节点**视图下载用于访问节点的SSH密钥。选择要访问的节点，然后单击行末的垂直省略号按钮，然后选择**下载密钥**，如下图所示:

![下载Keys]({{< baseurl >}}/img/rancher/downloadsshkeys.png)

解压缩下载的zip文件，并使用文件`id_rsa`连接到你的主机。一定要使用正确的用户名(`rancher` for RancherOS, `ubuntu` for Ubuntu, `ec2-user` for Amazon Linux)

```bash
ssh -i id_rsa user@ip_of_node
```
