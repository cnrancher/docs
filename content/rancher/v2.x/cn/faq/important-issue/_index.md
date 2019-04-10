---
title: 1 - 常见问题
weight: 1
---

## 1、何为PEM格式？

PEM格式通常用于数字证书认证机构(Certificate Authorities，CA)，扩展名为`.pem, .crt, .cer, and .key`。内容为Base64编码的ASCII码文件，有类似`"-----BEGIN CERTIFICATE-----" 和 "-----END CERTIFICATE-----"`的头尾标记。服务器认证证书，中级认证证书和私钥都可以储存为PEM格式(认证证书其实就是公钥)。Apache和类似的服务器使用PEM格式证书。

你可以通过以下特征识别PEM格式:

  ```bash
  - 该文件以下列标题开头:
  -----BEGIN CERTIFICATE-----
  - 标题后面跟着一串长字符
  - 该文件以页脚结尾:
  -----END CERTIFICATE-----
  ```

**PEM证书例如:**

  ```bash
  ----BEGIN CERTIFICATE-----
  MIIGVDCCBDygAwIBAgIJAMiIrEm29kRLMA0GCSqGSIb3DQEBCwUAMHkxCzAJBgNV
  ... more lines
  VWQqljhfacYPgp8KJUJENQ9h5hZ2nSCrI+W00Jcw4QcEdCI8HL5wmg==
  -----END CERTIFICATE-----
  ```

## 2、如果我想添加我的中间证书，证书的顺序是什么？

添加证书的顺序如下:

```bash
-----BEGIN CERTIFICATE-----
%YOUR_CERTIFICATE%
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
%YOUR_INTERMEDIATE_CERTIFICATE%
-----END CERTIFICATE-----
```

## 3、我如何验证我的证书链？

你可以使用`openssl`二进制验证证书链。如果该命令的输出(参见下面的命令示例)结束`Verify return code: 0 (ok)`，那么证书链是有效的。该`ca.pem`文件必须与你添加到`rancher/rancher`容器中的文件相同。当使用由认可的认证机构签署的证书时，可以省略该`-CAfile`参数。

**命令:**

```bash
openssl s_client -CAfile ca.pem -connect rancher.yourdomain.com:443
...
Verify return code: 0 (ok)
```

## 4、持久数据

Rancher `etcd`用作数据存储，使用单节点安装时，将使用内置`etcd`。持久数据位于容器中的以下路径中: `/var/lib/rancher`。你可以将主机卷挂载到此位置以保留其运行的数据。

**命令**:

```bash
# 指定主机路径
  HOST_PATH=xxxx
  docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v $HOST_PATH:/var/lib/rancher \
  rancher/rancher:latest
```

## 5、如何在同一个主机上运行`Rancher/Rancher`和`Rancher/Rancher-Agent`

在你想要使用单个节点运行Rancher并且能够将相同节点添加到集群的情况下，你必须调整为`rancher/rancher`容器映射的主机端口。

如果一个节点被添加到集群，它将部署使用端口80和443的ingress控制器。这与`rancher/rancher`容器默认映射的端口冲突。

>**注意**不建议在生产中把Rancher/Rancher和Rancher/Rancher-Agent运行在一台主机上，但可用于开发/演示。

要更改主机端口映射，替换`-p 80:80 -p 443:443`为`-p 8080:80 -p 8443:443`:

```bash
docker run -d --restart=unless-stopped \
  -p 8080:80 -p 8443:443 \
  -v <主机路径>:/var/lib/rancher/ \
  rancher/rancher:latest
```

## 6、如何重置管理员密码？

- 单节点安装

    ```bash
    docker exec -ti <container_id> reset-password
    New password for default admin user (user-xxxxx):
    <new_password>
    ```

- HA安装(Helm)

    ```bash
    KUBECONFIG=./kube_config_rancher-cluster.yml
    kubectl --kubeconfig $KUBECONFIG -n cattle-system exec $(kubectl --kubeconfig $KUBECONFIG -n cattle-system get pods -l app=rancher | grep '1/1' | head -1 | awk '{   print $1 }') -- reset-password
    New password for default admin user (user-xxxxx):
    <new_password>
    ```

- HA安装(RKE)

    ```bash
    KUBECONFIG=./kube_config_rancher-cluster.yml
    kubectl --kubeconfig $KUBECONFIG exec -n cattle-system \
    $(kubectl --kubeconfig $KUBECONFIG get pods -n cattle-system \
    -o json | jq -r '.items[] | \
    select(.spec.containers[].name=="cattle-server") | \
    .metadata.name') -- reset-password

    New password for default admin user (user-xxxxx):
    <new_password>
    ```

## 7、我删除/停用了管理员，我该如何恢复？

- 单节点安装

      ```bash
      docker exec -ti <container_id> ensure-default-admin
      New default admin user (user-xxxxx)
      New password for default admin user (user-xxxxx):
      <new_password>
      ```
- HA安装(Helm)

    ```bash
    KUBECONFIG=./kube_config_rancher-cluster.yml
    kubectl --kubeconfig $KUBECONFIG -n cattle-system exec $(kubectl --kubeconfig $KUBECONFIG -n cattle-system get pods -l app=rancher | grep '1/1' | head -1 | awk '{ print $1 }') -- ensure-default-admin
    New password for default admin user (user-xxxxx):
    <new_password>
    ```

- HA安装(RKE)

    ```bash
    KUBECONFIG=./kube_config_rancher-cluster.yml
    kubectl --kubeconfig $KUBECONFIG exec -n cattle-system \
    $(kubectl --kubeconfig $KUBECONFIG get pods -n cattle-system \
    -o json | jq -r '.items[] | select(.spec.containers[].name=="cattle-server") | \
    .metadata.name') -- ensure-default-admin

    New password for default admin user (user-xxxxx):
    <new_password>
    ```

## 8、怎么样开启debug模式？

### 单节点安装

- 启用

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

### HA安装(RKE)

- 启用

    ```bash
    KUBECONFIG=./kube_config_rancher-cluster.yml
    kubectl --kubeconfig $KUBECONFIG exec -n cattle-system \
    $(kubectl --kubeconfig $KUBECONFIG get pods -n cattle-system \
    -o json | jq -r '.items[] | select(.spec.containers[].name=="cattle-server") \
    | .metadata.name') -- loglevel --set debug
    OK
    kubectl --kubeconfig $KUBECONFIG logs -n cattle-system -f \
    $(kubectl --kubeconfig $KUBECONFIG get pods -n cattle-system \
    -o json | jq -r '.items[] | select(.spec.containers[].name="cattle-server") | \
    .metadata.name')
    ```

- 禁用

    ```bash
    KUBECONFIG=./kube_config_rancher-cluster.yml
    kubectl --kubeconfig $KUBECONFIG exec -n cattle-system \
    $(kubectl --kubeconfig $KUBECONFIG get pods -n cattle-system \
    -o json | jq -r '.items[] | select(.spec.containers[].name=="cattle-server") | \
    .metadata.name') -- loglevel --set info
    OK
    ```

### HA安装(Helm)

- 启用

    ```bash
    KUBECONFIG=./kube_config_rancher-cluster.yml

    kubectl --kubeconfig $KUBECONFIG -n cattle-system \
    get pods -l app=rancher | grep '1/1' | awk '{ print $1 }' | \
    xargs -I{} kubectl --kubeconfig $KUBECONFIG -n cattle-system \
    exec {} -- loglevel --set debug

    kubectl --kubeconfig $KUBECONFIG -n cattle-system logs -l app=rancher
    ```

- 禁用

    ```bash
    KUBECONFIG=./kube_config_rancher-cluster.yml

    kubectl --kubeconfig $KUBECONFIG -n cattle-system \
    get pods -l app=rancher | grep '1/1' | awk '{ print $1 }' | \
    xargs -I{} kubectl --kubeconfig $KUBECONFIG -n cattle-system exec {} \
    -- loglevel --set info

    ```

## 9、ClusterIP无法ping通？

ClusterIP是一个虚拟IP，不会响应ping。测试ClusterIP配置是否正确的最好方法是使用`curl`来访问IP和端口以查看它是否响应。

## 10、我在哪里可以管理主机模板？

打开你的帐户菜单(右上角)，并选择`主机模板`。

## 11、为什么我的L4层负载均衡服务处于“挂起”状态？

L4层负载均衡器创建为`type:LoadBalancer`，在Kubernetes中，这需要云提供商或控制器能够满足这些请求，否则这些将永远处于“挂起”状态。 了解更多[云提供商]({{< baseurl >}}/rancher/v2.x/cn/concepts/clusters/cloud-providers/) 或者 [Create External Load Balancer](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/)

## 12、Rancher的状态存储在什么地方？

- 单节点安装

  在rancher/rancher容器的内置etcd中，映射与宿主机的`/var/lib/rancher`目录下。

- HA安装

  RKE部署集群指定的ETCD中，默认与Kubernetes共有一套ETCD服务。

## 13、如何确定支持的Docker版本？

我们遵循经过Kubernetes官方验证过的Docker版本，已验证的Docker版本可以在Kubernetes的[发版记录](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.10.md#external-dependencies)中找到。

## 14、我如何访问Rancher创建的节点？

可以通过**节点**视图下载用于访问节点的SSH密钥。选择要访问的节点，然后单击行末的垂直省略号按钮，然后选择**下载密钥**，如下图所示:

![下载Keys]({{< baseurl >}}/img/rancher/downloadsshkeys.png)

解压缩下载的zip文件，并使用文件`id_rsa`连接到你的主机。一定要使用正确的用户名(`rancher` for RancherOS, `ubuntu` for Ubuntu, `ec2-user` for Amazon Linux)

```bash
ssh -i id_rsa user@ip_of_node
```

## 15、如何在Rancher中自动执行任务

UI由静态文件组成，并且基于API的响应而工作。这意味着您可以在UI中执行的每个操作/任务都可以通过API自动执行。有两种方法可以做到这一点：

- 访问`https://your_rancher_ip/v3`并浏览API选项。
- 使用用户界面时捕获API调用（最常用的是[Chrome开发者工具，](https://developers.google.com/web/tools/chrome-devtools/#network)您可以使用您喜欢的浏览器）

## 16、节点的IP地址发生了变化，我该如何恢复？

节点需要配置静态IP（或通过DHCP保留IP）。如果节点的IP已更改，则必须将其从集群中删除。删除后，Rancher会将集群更新为正确的状态。如果集群不再处于`Provisioning`状态，则已经从集群中删除该节点。当节点的IP地址发生变化时，Rancher失去了与节点的连接，因此无法正常清理节点。请参阅[清理集群节点]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/remove-node/)以清除节点。

从集群中删除节点并清除节点后，您可以将节点重新添加到集群。

## 17、如何在Rancher安装的Kubernetes集群中向Kubernetes组件添加额外的arguments/binds/environment？

可以通过`集群选项`中的[配置文件]({{< baseurl >}}/rancher/v2.x/cn/cluster-provisioning/rke-clusters/options/#配置文件)选项添加额外的arguments/binds/environment。有关更多信息，请参阅RKE文档中的[Extra Args，Extra Binds和Extra Environment Variables]({{< baseurl >}}/rke/v0.1.x/en/config-options/services/services-extras/)，或浏览[Example Cluster.ymls示例]({{< baseurl >}}/rke/v0.1.x/en/example-yamls/)。

## 18、为什么在节点出现故障时重新调度pod需要5分钟以上？

这是由于以下默认Kubernetes设置的组合：

- kubelet

  - `node-status-update-frequency`：指定kubelet将节点状态发布到master的频率（默认为10秒）
- kube-controller-manager
  - `node-monitor-period`：在NodeController中同步NodeStatus的时间段（默认5秒）
  - `node-monitor-grace-period`：在标记运行节点不健康之前允许运行节点无响应的时间（默认为40秒）
  - `pod-eviction-timeout`：删除失败节点上的pod的宽限期（默认为5m0）

    有关这些设置的更多信息，请参阅[Kubernetes：kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)和[Kubernetes：kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)。

## 19、如何通过`证书`查询`Common Name`和`Subject Alternative Names`？

- 检查`Common Name`：

    ```bash
    openssl x509 -noout -subject -in cert.pem
    subject=/CN=rancher.my.org
    ```

- 检查`Subject Alternative Names`：

    ```bash
    openssl x509 -noout -in cert.pem -text | grep DNS
    DNS: rancher.my.org
    ```