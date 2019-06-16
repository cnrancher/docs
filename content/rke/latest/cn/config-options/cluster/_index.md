---
title: 2 - Kubernetes集群
weight: 2
---

## 一、集群名称

默认情况下，集群的名称将是`local`。如果需要其他名称，可以使用`cluster_name`更改集群的名称,该名称将在集群生成的`kubeconfig`文件中显示。

```yaml
cluster_name: mycluster
```

## 二、支持的Docker版本

默认情况下，RKE将检查所有主机上已安装的Docker版本，如果Kubernetes不支持该版本，则会失败并显示错误。[支持的Docker版本](https://github.com/rancher/rke/blob/master/docker/docker.go#L37-L41)列表专门针对每个Kubernetes版本设置。默认值为`false`。

```yaml
ignore_docker_version: true
```

## 三、Kubernetes版本

默认情况下，RKE使用特定的Kubernetes版本启动，您可以选择为集群安装的不同版本的Kubernetes。每个版本的RKE都有一个支持的Kubernetes版本的特定列表。

您可以按如下方式设置Kubernetes版本：

```yaml
kubernetes_version: "v1.11.6-rancher1-1"
```

如果同时设置了`kubernetes_version`和[system images]({{< baseurl >}}/rke/latest/cn/config-options/system-images/)，则优先使用`system images`。

### 1、列出支持的Kubernetes版本

请参阅您当前运行的RKE版本的[release notes](https://github.com/rancher/rke/releases)，以查找支持的Kubernetes版本列表以及默认的Kubernetes版本。也可以通过命令列出特定RKE版本下受支持的`kubernetes_version`和`system images`

```bash
$ rke config --system-images --all

INFO[0000] Generating images list for version [v1.13.4-rancher1-2]:
.......
INFO[0000] Generating images list for version [v1.11.8-rancher1-1]:
.......
INFO[0000] Generating images list for version [v1.12.6-rancher1-2]:
.......
```

### 2、使用不受支持的Kubernetes版本

- 从v0.2.0开始，如果`kubernetes_version`设置的版本不在支持的Kubernetes版本列表中，则RKE将报错；

- 在v0.2.0之前，如果`kubernetes_version`设置的版本不在支持的Kubernetes版本列表中，则使用当前rke版本支持的默认K8S版本。

- 如果要使用受支持列表中的其他版本，请使用[system images]({{< baseurl >}}/rke/latest/cn/config-options/system-images/)定义版本。

## 四、集群级SSH密钥路径

RKE使用`ssh`连接到主机。通常，每个节点将为每个ssh密钥设置一个独立路径，即在nodes部分中配置了`ssh_key_path`。如果所有节点使用相同的`ssh`私钥登录，您们可以在集群级别设置`ssh_key_path`。

如果在集群级别和节点级别都设置了`ssh_key_path`，则节点级别密钥将优先使用。

```yaml
ssh_key_path: ~/.ssh/test
```

## 五、SSH代理

RKE支持使用本地`ssh agent`进行ssh连接，此选项的默认值为`false`。如果要使用本地`ssh agent`进行ssh连接，则应将其设置为`true`。

```yaml
ssh_agent_auth: true
```

如果要使用`带密码的SSH私钥`，则需要将密钥添加到`ssh-agent`并配置`SSH_AUTH_SOCK`环境变量。

```bash
$ eval "$(ssh-agent -s)"
Agent pid 3975
$ ssh-add /home/user/.ssh/id_rsa
Enter passphrase for /home/user/.ssh/id_rsa:
Identity added: /home/user/.ssh/id_rsa (/home/user/.ssh/id_rsa)
$ echo $SSH_AUTH_SOCK
/tmp/ssh-118TMqxrXsEx/agent.3974
```

## 六、附加组件加载超时时间

您可以定义在Kubernetes集群正常运行之后，通过Kubernetes [jobs](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/)部署附加服务。配置job的超时时间，在超时后，RKE将停止尝试检索job状态。以秒为单位。默认超时值为`30`秒。

```yaml
addon_job_timeout: 30
```

## 七、卷映射路径前缀(Prefix Path)

`rke`在创建集群时，会把主机的`/etc/kubernetes，/var/lib/etcd`等目录映射到容器中，以供容器调用相关配置文件。但对于像`RancherOS和CoreOS`这样的操作系统，没有权限直接在`根路径`写入文件，因此需要指定一个子目录创建相关配置文件。

```bash
prefix_path: /custom_path
```

这样在创建好集群后，容器映射的路径格式为：

```bash
-v /custom_path/etc/kubernetes:/etc/kubernetes
```
