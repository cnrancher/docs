---
title: 3 - RKE HA升级
weight: 3
---

>此方法仅适用于Rancher:v2.0.8及之前的版本

## 先决条件

从v2.0.7开始，Rancher引入了`system`项目，该项目是自动创建的，用于存储Kubernetes需要运行的重要命名空间。在升级到`v2.0.7+`前，请检查环境中有没有创建`system`项目，如果有则删除。`并检查确认所有系统命名空间未分配到任何项目下，如果有则移到出去，以防止群集网络问题。`

- Rancher Kubernetes Engine v0.1.7或更高版本

    etcd快照功能仅在RKE v0.1.7及更高版本中可用

- 保证所有ETCD节点具有相同的快照版本

- `kubectl安装`

    在主机或者远程访问的笔记本上安装[kubectl]({{< baseurl >}}/rancher/v2.x/cn/installation/download/)命令行工具

- `rancher-cluster.yml(RKE配置文件)`

    通过RKE创建kubernetes集群，需要预先设置rancher-cluster.yml配置文件，通过这个配置文件安装kubernetes集群，这个文件需要与RKE二进制文件存放同一目录。

- 确认系统存在以下路径:~/.kube/，如果没有，请自行创建。

- `kube_config_rancher-cluster.yml`(kubectl配置文件)

    RKE安装kubernetes集群后，会在RKE二进制文件相同目录下生成`kube_config_rancher-cluster.yml`文件，复制该配置文件到`~/.kube/`目录.

### 操作步骤

1. 在安装了kubectl命令行工具的电脑上打开终端

2. 切换路径到RKE二进制文件所在目录，确认rancher-cluster.yml在同一路径下

3. 创建ETCD快照备份

    替换`<SNAPSHOT.db>`为你喜欢的快照名称(例如upgrade.db)

    ```bash
    # MacOS
    ./rke_darwin-amd64 etcd snapshot-save --name <SNAPSHOT.db> --config rancher-cluster.yml
    # Linux
    ./rke_linux-amd64 etcd snapshot-save --name <SNAPSHOT.db> --config rancher-cluster.yml
    ```
    > RKE获取每个etcd节点上的运行快照，保存快照文件当前到etcd节点的`/opt/rke/etcd-snapshots`目录下.

4. Rancher 升级

    输入以下命令进行升级:

    ```bash
    kubectl --kubeconfig=kube_config_rancher-cluster.yml set image deployment/cattle cattle-server=rancher/rancher:<VERSION_TAG> -n cattle-system
    ```
    替换`<VERSION_TAG>`为想要升级到的版本，可用的镜像版本可查阅[DockerHub](https://hub.docker.com/r/rancher/rancher/tags/)。

    >请勿使用后缀为rc或者为master的镜像，具体详情请查看[版本标签]({{< baseurl >}}/rancher/v2.x/cn/installation/server-tags/)。

5. 登录Rancher UI,通过检查浏览器窗口左下角显示的版本，确认是否升级成功。
