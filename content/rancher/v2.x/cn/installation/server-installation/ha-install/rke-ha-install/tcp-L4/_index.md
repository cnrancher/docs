---
title: 1 - 四层负载均衡HA部署
weight: 1
---
>### **重要提示:**
>RKE HA安装仅支持Rancher v2.0.8以及之前的版本，Rancher v2.0.8之后的版本使用[helm安装Rancher]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install/helm-rancher/)。

以下步骤将创建一个新的Kubernetes集群，专用于Rancher高可用(HA)运行，本文档将引导你使用Rancher Kubernetes Engine(RKE)配置三个节点的集群。

## 一、架构说明

![Rancher HA]({{< baseurl >}}/img/rancher/ha/rancher2ha.svg)

## 一、Linux主机要求

- [基础环境配置]({{< baseurl >}}/rancher/v2.x/cn/installation/basic-environment-configuration/)
- [端口需求]({{< baseurl >}}/rancher/v2.x/cn/installation/references/)

## 二、配置负载均衡器(以NGINX为例)

我们将使用NGINX作为第4层负载均衡器(TCP)。NGINX会将所有连接转发到你的Rancher节点之一。如果要使用Amazon NLB，可以跳过此步骤并使用[Amazon NLB configuration]({{< baseurl >}}/rancher/v2.x/cn/installation/server-installation/ha-install-external-lb/tcp-l4/amazon-nlb-configuration/)配置。

>**注意:** 在此配置中，负载平衡器位于Rancher节点的前面，负载均衡器可以是任意能够运行NGINX的主机。`不要使用任意一个Rancher节点作为负载均衡器节点`,会出现端口冲突。

### 1、安装nginx

首先在负载均衡器主机上安装NGINX，NGINX具有适用于所有已知操作系统的软件包。有关安装NGINX的帮助，请查阅其安装文档[nginx安装文档](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/).

### 2、创建NGINX配置

安装NGINX后，你需要使用节点的IP地址更新NGINX配置文件`nginx.conf`。

1. 复制下面的代码到文本编辑器，保存为`nginx.conf`。
2. 在`nginx.conf`配置中, 替换`IP_NODE_1`、`IP_NODE_2`、`IP_NODE_3` 为你主机真实的IP地址。

    **NGINX配置示例:**

    ```bash
    worker_processes 4;
    worker_rlimit_nofile 40000;

    events {
        worker_connections 8192;
    }
    http {
        server {
            listen         80;
            return 301 https://$host$request_uri;
        }
    }
    stream {
        upstream rancher_servers {
            least_conn;
            server IP_NODE_1:443 max_fails=3 fail_timeout=5s;
            server IP_NODE_2:443 max_fails=3 fail_timeout=5s;
            server IP_NODE_3:443 max_fails=3 fail_timeout=5s;
        }
        server {
            listen     443;
            proxy_pass rancher_servers;
        }
    }
    ```

3. 保存 `nginx.conf` ，并复制`nginx.conf`到负载均衡器节点的`/etc/nginx/nginx.conf`路径下。
4. 重新加载nginx配置

    ```bash
    nginx -s reload
    ```

### 3、可选 - 以容器运行nginx服务

我们可以以容器的形式运行nginx服务，而不需要把它安装在宿主机上。将编辑好的NGINX示例配置文件保存到/etc/nginx.conf，并运行以下命令来启动NGINX容器:

```bash
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /etc/nginx.conf:/etc/nginx/nginx.conf \
  nginx:1.14
```

## 三、配置DNS

选择一个用于访问Rancher的域名(FQDN)(例如: demo.rancher.com).

- 方案1 - 有DNS服务器

1. 登录DNS服务，创建一条 `A` 记录指向负载均衡主机IP。

2. 在终端中执行一下命令来验证运行解析是否生效:

    `nslookup HOSTNAME.DOMAIN.COM`

    - 如果解析生效:

        ```bash
        nslookup demo.rancher.com
        DNS Server:         YOUR_HOSTNAME_IP_ADDRESS
        DNS Address:        YOUR_HOSTNAME_IP_ADDRESS#53
        Non-authoritative answer:
        Name:   demo.rancher.com
        Address: <负载均衡IP地址>
        ```
    - 如果解析不生效

        ```bash
        nslookup demo.rancher.com
        DNS Server:         YOUR_HOSTNAME_IP_ADDRESS
        DNS Address:        YOUR_HOSTNAME_IP_ADDRESS#53

        ** server can't find demo.rancher.com: NXDOMAIN
        ```

- 方案2 - 无DNS服务器

1. 如果环境为内部网络且无DNS服务器，可以通过修改客户端的`/etc/hosts`文件，添加相应的条目。例如:

    ![image-20180711140926370]({{< baseurl >}}/img/rancher/ha/image-20180711140926370.png)

## 四、下载 RKE

RKE是一种快速，通用的Kubernetes安装程序，可用于在Linux主机上安装Kubernetes。我们将使用RKE来配置Kubernetes集群并运行Rancher。

1、访问 [文件下载]({{< baseurl >}}/rancher/v2.x/cn/installation/download/) 页面，根据你操作系统类型下载最新版本的RKE:

- **MacOS**: `rke_darwin-amd64`
- **Linux**: `rke_linux-amd64`
- **Windows**: `rke_windows-amd64.exe`

2、通过`chmod +x`命令给刚下载的RKE二进制文件添加可执行权限。

>如果是Windows系统，则跳过这一步.

```bash
# MacOS
$ chmod +x rke_darwin-amd64
# Linux
$ chmod +x rke_linux-amd64
```

3、确认RKE是否是最新版本:

```bash
# MacOS
./rke_darwin-amd64 --version
# Linux
./rke_linux-amd64 --version
```

**结果:** 你将看到以下内容:

```bash
rke version v<N.N.N>
```

## 五、下载RKE配置模板

RKE通过 `.yml` 配置文件来安装和配置Kubernetes集群，有2个模板可供选择，具体取决于使用的SSL证书类型。

1、根据你使用的SSL证书类型，选择模板下载

- [3-node-certificate.yml](https://raw.githubusercontent.com/rancher/rancher/e9d29b3f3b9673421961c68adf0516807d1317eb/rke-templates/3-node-certificate.yml)

- [3-node-certificate-recognizedca.yml](https://raw.githubusercontent.com/rancher/rancher/d8ca0805a3958552e84fdf5d743859097ae81e0b/rke-templates/3-node-certificate-recognizedca.yml)

2、重命名模板文件为 `rancher-cluster.yml`

## 六、节点配置

1、节点免密登录

- 第一步:在任意一台Linux主机使用ssh-keygen命令产生公钥私钥对

    ```bash
    ssh-keygen
    ```

- 第二步:通过ssh-copy-id命令将公钥复制到远程机器中

    ```bash
    ssh-copy-id -i .ssh/id_rsa.pub  $user@192.168.x.xxx
    ```

2、编辑`rancher-cluster.yml`配置

编辑器打开 `rancher-cluster.yml` 文件,在nodes配置版块中，修改 `IP_ADDRESS_X` and `USER`为你真实的Linux主机IP和用户名,`ssh_key_path`为第一步生成的私钥文件，如果是在RKE所在主机上生成的公钥私钥对，此配置可保持默认:

```yaml
nodes:
  - address: `IP_ADDRESS_1`
    user: `USER`
    role: [controlplane,etcd,worker]
    ssh_key_path: ~/.ssh/id_rsa
  - address: `IP_ADDRESS_2`
    user: `USER`
    role: [controlplane,etcd,worker]
    ssh_key_path: ~/.ssh/id_rsa
  - address: `IP_ADDRESS_3`
    user: `USER`
    role: [controlplane,etcd,worker]
    ssh_key_path: ~/.ssh/id_rsa
```

>**注意**
>1、使用RHEL/CentOS系统时，因为系统安全限制，`ssh`不能使用root账户。\
>2、需要开启[API审计日志？]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/api-auditing/)\
>3、了解RKE[配置参数]({{< baseurl >}}/rke/v0.1.x/en/config-options/)

## 七、证书配置

出于安全考虑，使用Rancher需要SSL加密。 SSL可以保护所有Rancher网络通信，例如登录或与集群交互时。

- 方案A — 使用自签名证书

    >**先决条件:** 1.证书必须是`PEM格式`,`PEM`只是一种证书类型，并不是说文件必须是PEM为后缀，具体可以查看[证书类型]({{< baseurl >}}/rancher/v2.x/cn/installation/self-signed-ssl/)；\
    > 2.证书必须通过`base64`加密；\
    > 3.在你的证书文件中，包含链中的所有中间证书；

1. 在`kind: Secret`与`name: cattle-keys-ingress`中:

  - 替换 `<BASE64_CRT>` 为证书文件经过base64加密的字符串(证书文件通常名为 `cert.pem` 或 `domain.crt`)
  - 替换 `<BASE64_KEY>` 为证书密钥文件经过base64加密的字符串(通过证书密钥文件名为`key.pem`或`domain.key`)

    >**注意:** base64编码的字符串应与tls.crtor&tls.key在同一行，并且在开头，冒号后有一个空格，中间或末尾没有任何换行符。

    **结果:** 替换值后，文件应如下所示(base64编码的字符串应该不同)

    ```yaml
    ---
      apiVersion: v1
      kind: Secret
      metadata:
        name: cattle-keys-ingress
        namespace: cattle-system
      type: Opaque
      data:
        tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1RENDQWN5Z0F3SUJBZ0lKQUlHc25NeG1LeGxLTUEwR0
        tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBdEY3WEN6TVZHaDF1aU5oWTBJZW
    ```

2. 在`kind: Secret` 和 `name: cattle-keys-server`中, 替换<BASE64_CA>为CA证书文件的base64加密字符串(通常称为ca.pem或ca.crt)。

    >**注意:** base64编码的字符串应该与cacerts.pem在同一行，冒号后一个空格，在开头，中间或结尾没有任何换行符。

    **结果:** 该文件修改后应如下所示(base64编码的字符串应该不同):

    ```yaml
    ---
    apiVersion: v1
    kind: Secret
    metadata:
      name: cattle-keys-server
      namespace: cattle-system
    type: Opaque
    data:
      cacerts.pem: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNvRENDQVlnQ0NRRHVVWjZuMEZWeU
    ```

- 方案B — 使用权威CA机构颁发的SSL证书

    如果你使用的是由权威CA机构颁发的SSL证书，则需要为证书文件和证书密钥文件生成base64编码的字符串(确保你的证书文件包含链中的所有中间证书)。在这种情况下，证书的顺序首先是你自己的证书，然后是中间证书。请查阅CSP(证书服务提供商)的文档，了解需要包含哪些中间证书。

1. 在`kind: Secret`和 `name: cattle-keys-ingress`中:

  - 替换`<BASE64_CRT>`为证书文件的base64加密字符串(通常称为`cert.pem`或`domain.crt`);

  - 替换`<BASE64_CRT>`为证书文件的base64加密字符串(通常称为`cert.pem`或`domain.crt`);

    替换值后，该文件应如下所示(base64编码的字符串应该不同):

    >**注意:** base64编码的字符串应与tls.crtor 在同一行tls.key，冒号后有一个空格，并且在开头，中间或末尾没有任何换行符。

    ```yaml
    ---
    apiVersion: v1
    kind: Secret
    metadata:
      name: cattle-keys-ingress
      namespace: cattle-system
    type: Opaque
    data:
      tls.crt:
      tls.key:
    ```

## 八、域名配置

配置文件中有两个引用了`<FQDN>`，一个是在spec\rules\host处，tls\hosts处。

在`kind: Ingress` 和 `name: cattle-ingress-http`中，替换 `<FQDN>`为预先准备的域名,替换后应为如下显示:

```yaml
 ---
  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    namespace: cattle-system
    name: cattle-ingress-http
    annotations:
      nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
      nginx.ingress.kubernetes.io/proxy-read-timeout: "1800"   # Max time in seconds for w  to remain shell        window open
      nginx.ingress.kubernetes.io/proxy-send-timeout: "1800"   # Max time in seconds for w  to remain shell        window open
  spec:
    rules:
    - host: demo.rancher.com
      http:
        paths:
        - backend:
            serviceName: cattle-service
            servicePort: 80
    tls:
    - secretName: cattle-keys-ingress
      hosts:
      - demo.rancher.com
```

## 九、备份配置文件

保存关闭.yml文件后，将其备份到安全位置。升级Rancher时，你需要再次使用此文件。

## 十、运行RKE

完成所有配置后，你可以通过运行rke up命令并使用--config参数指定配置文件来完成Rancher 集群的安装。

1、下载RKE二进制文档到你的主机，确保 `rancher-cluster.yml`与下载的`rke` 在同一目录下；

2、打开shell 终端，切换路径到RKE所在的目录；

3、根据操作系统类型，选择以下命令并执行:

  ```bash
  # MacOS
  ./rke_darwin-amd64 up --config rancher-cluster.yml
  # Linux
  ./rke_linux-amd64 up --config rancher-cluster.yml
  ```

  **结果:** 应该会有以下日志输出:

  ```yaml
  INFO[0000] Building Kubernetes cluster
  INFO[0000] [dialer] Setup tunnel for host [1.1.1.1]
  INFO[0000] [network] Deploying port listener containers
  INFO[0000] [network] Pulling image [alpine:latest] on host [1.1.1.1]
  ...
  INFO[0101] Finished building Kubernetes cluster successfully
  ```

## 十一、备份自动生成的kubectl配置文件

在安装过程中，RKE会自动生成一个kube_config_rancher-cluster.yml与RKE二进制文件位于同一目录中的配置文件。此文件很重要，它可以在Rancher server故障时，利用kubectl通过此配置文件管理Kubernetes集群。复制此文件将其备份到安全位置。

## 十二、(可选)为Agent Pod添加主机别名(/etc/hosts)

如果你没有内部DNS服务器而是通过添加`/etc/hosts`主机别名的方式指定的Rancher server域名，那么不管通过哪种方式(自定义、导入、Host驱动等)创建K8S集群，K8S集群运行起来之后，因为`cattle-cluster-agent Pod`和`cattle-node-agent`无法通过DNS记录找到`Rancher server`,最终导致无法通信。

### 解决方法

可以通过给`cattle-cluster-agent Pod`和`cattle-node-agent`添加主机别名(/etc/hosts)，让其可以正常通信`(前提是IP地址可以互通)`。

1. cattle-cluster-agent pod

    ```bash
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml #指定kubectl配置文件
    kubectl -n cattle-system patch  deployments cattle-cluster-agent --patch '{
        "spec": {
            "template": {
                "spec": {
                    "hostAliases": [
                        {
                            "hostnames":
                            [
                                "demo.cnrancher.com"
                            ],
                                "ip": "192.168.1.100"
                        }
                    ]
                }
            }
        }
    }'
    ```

2. cattle-node-agent pod

    ```bash
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml #指定kubectl配置文件
    kubectl -n cattle-system patch  daemonsets cattle-node-agent --patch '{
        "spec": {
            "template": {
                "spec": {
                    "hostAliases": [
                        {
                            "hostnames":
                            [
                                "xxx.rancher.com"
                            ],
                                "ip": "192.168.1.100"
                        }
                    ]
                }
            }
        }
    }'
    ```
    > **注意**
    >1、替换其中的域名和IP \
    >2、别忘记json中的引号。

## 十三、FAQ和故障排除

[FAQ]({{< baseurl >}}/rancher/v2.x/cn/faq/)中整理了常见的问题与解决方法。
