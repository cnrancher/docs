---
title: 6 - 管理员密码重置
weight: 6
---

## Rancher单节点安装

在Rancher运行的主机上，执行以下命令：

```bash
docker exec -ti <container_id> reset-password
```

显示结果：

```bash
New password for default admin user (user-xxxxx):
  <new_password>
```

## RancherHA安装

在安装有`kubectl`主机上，指定kubectl配置文件，然后运行以下命令：
>假设kubectl配置文件在当前目录下

```bash
KUBECONFIG=./kube_config_rancher-cluster.yml
  kubectl --kubeconfig $KUBECONFIG exec -n cattle-system \
  $(kubectl --kubeconfig $KUBECONFIG get pods -n cattle-system \
  -o json | jq -r '.items [] | \
  select(.spec.containers[].name=="cattle-server") | .metadata.name') \
  --reset-password
```

运行结果：

```bash
New password for default admin user (user-xxxxx):
  <new_password>
```