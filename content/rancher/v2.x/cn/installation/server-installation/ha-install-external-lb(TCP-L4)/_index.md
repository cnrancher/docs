---
title: HA安装+外部四层负载(TCP-L4)
weight: 275
---
以下步骤将创建一个新的Kubernetes集群，专用于Rancher高可用(HA)运行。本文档将引导您使用Rancher Kubernetes Engine(RKE)配置三个节点的集群,群集的唯一目的是为了以POD来运行Rancher.

![Rancher HA]({{< baseurl >}}/img/rancher/ha/rancher2ha.svg)

## 一、架构说明

## 二、配置Linux主机

在安装Rancher之前，请确认您符合主机要求。使用以下要求配置3个新的Linux主机。

### 要求：

1. #### 操作系统

    - Ubuntu 16.04（64位）
    - 红帽企业Linux 7.5（64位）
    - RancherOS 1.3.0（64位）

2. #### 硬件

    硬件需求根据Rancher部署的规模进行扩展。根据需求配置每个节点。

    | 部署大小 | 集群(个)  | 节点(个) | vCPU                                            | 内存 |
    | -------- | --------- | -------- |     ------------------------------------------- | ---- |
    | 小       | 不超过10  | 最多50   | 2C                                              | 4GB  |
    | 中       | 不超过100 | 最多500  | 8C                                              | 32GB |
    | 大       | 超过100   | 超过500  | [联系Rancher](https://rancher.com/contact/) |      |

3. #### 软件

    - Docker

      > **注意：**如果您使用的是RancherOS，请确保您将Docker引擎切换为受支持的版本`sudo ros engine switch docker-17.03.2-ce`

      **支持的Docker版本**

      - `1.12.6`
      - `1.13.1`
      - `17.03.2`

      [Docker安装说明](https://docs.docker.com/install/)

      > **注意：** 该`rancher/rancher`镜像托管在[DockerHub上](https://hub.docker.com/r/rancher/rancher/tags/)。如果您无法访问DockerHub，或者离线环境下安装Rancher，请参阅[Air Gap安装](/docs/rancher/v2.x/cn/installation/server-installation/air-gap-installation/)。
      >
      > 有关可用的其他Rancher server标记的列表，请参阅[Rancher server tags](/docs/rancher/v2.x/cn/installation/server-tags/)。

4. #### 端口

    下图描述了Rancher的基本端口要求。有关全面列表，请参阅[端口要求](/docs/rancher/v2.x/cn/installation/references/)。

      ![基本端口要求](/docs/img/rancher/port-communications.png)

## 三、配置负载均衡器(以NGINX为例)

我们将使用NGINX作为第4层负载均衡器(TCP)。NGINX会将所有连接转发到您的Rancher节点之一。如果要使用Amazon NLB，可以跳过此步骤并使用[Amazon NLB configuration]({{< baseurl >}}/rancher/v2.x/en/installation/ha-server-install/nlb/)配置。

>**注意:**
> 在此配置中，负载平衡器位于Linux主机的前面。负载均衡器可以是任何能够运行NGINX的主机。
>
>一个警告：不要使用其中一个Rancher节点作为负载均衡器。

1. ### 安装nginx

    首先在负载均衡器主机上安装NGINX。NGINX具有适用于所有已知操作系统的软件包。有关安装NGINX的帮助，请参阅其安装文档。[nginx安装文档](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/).

2. ### 创建NGINX配置

    安装NGINX后，您需要使用节点的IP地址更新NGINX配置文件`nginx.conf`。

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

3. ### 可选 - 以容器运行nginx服务

    我们可以以容器的形式运行nginx服务，而不需要把它安装在宿主机上。将编辑好的NGINX示例配置文件保存到/etc/nginx.conf，并运行以下命令来启动NGINX容器：

    ```bash
    docker run -d --restart=unless-stopped \
      -p 80:80 -p 443:443 \
      -v /etc/nginx.conf:/etc/nginx/nginx.conf \
      nginx:1.14
    ```

## 四、配置DNS

选择一个用于访问Rancher的域名(FQDN)(例如，demo.rancher.com).

- 方案1 - 有DNS服务器

1. 登录DNS服务，创建一条 `A` 记录指向负载均衡主机IP。

2. 在终端中执行一下命令来验证运行解析是否生效:

    `nslookup HOSTNAME.DOMAIN.COM`

    1. 如果解析生效：

        ```bash
        nslookup rancher.yourdomain.com
        DNS Server:         YOUR_HOSTNAME_IP_ADDRESS
        DNS Address:        YOUR_HOSTNAME_IP_ADDRESS#53    
        Non-authoritative answer:
        Name:   demo.rancher.com
        Address: <负载均衡IP地址>
        ```
    2. 如果解析不生效
        ```bash
        nslookup demo.rancher.com
        DNS Server:         YOUR_HOSTNAME_IP_ADDRESS
        DNS Address:        YOUR_HOSTNAME_IP_ADDRESS#53

        ** server can't find demo.rancher.com: NXDOMAIN
        ```

## 五、下载 RKE

RKE是一种快速，通用的Kubernetes安装程序，可用于在Linux主机上安装Kubernetes。我们将使用RKE来配置Kubernetes集群并运行Rancher。

1. 打开浏览器访问 [RKE Releases](https://github.com/rancher/rke/releases/latest) 页面，根据你操作系统类型下载最新版本的RKE：

    - **MacOS**: `rke_darwin-amd64`
    - **Linux**: `rke_linux-amd64`
    - **Windows**: `rke_windows-amd64.exe`

2. 通过`chmod +x`命令给刚下载的RKE二进制文件添加可执行权限。

    >如果是Windows系统，则跳过一步，直接进入[下载RKE配置模板](#下载RKE配置模板).

    ```bash
    # MacOS
    $ chmod +x rke_darwin-amd64
    # Linux
    $ chmod +x rke_linux-amd64
    ```

3. 确认RKE是否是最新版本：

    ```bash
    # MacOS
    $ ./rke_darwin-amd64 --version
    # Linux
    $ ./rke_linux-amd64 --version
    ```

    **Step Result:** You receive output similar to what follows:

    ```bash
    rke version v<N.N.N>
    ```

## 六、下载RKE配置模板

RKE通过 `.yml` 配置文件来安装和配置Kubernetes集群，有2个模板可供选择，具体取决于使用的SSL证书类型。

1. 根据您使用的SSL证书类型，选择模板下载：

    - [3-node-certificate.yml](https://raw.githubusercontent.com/rancher/rancher/e9d29b3f3b9673421961c68adf0516807d1317eb/rke-templates/3-node-certificate.yml)

    - [3-node-certificate-recognizedca.yml](https://raw.githubusercontent.com/rancher/rancher/d8ca0805a3958552e84fdf5d743859097ae81e0b/rke-templates/3-node-certificate-recognizedca.yml)

2. 重命名模板文件为 `rancher-cluster.yml`.

### 1. 节点配置

1. 编辑器打开 `rancher-cluster.yml` 文件；

2. 在nodes配置版块中，修改 `IP_ADDRESS_X` and `USER`为你真实的Linux主机IP和用户名,`ssh_key_path`可保持系统默认路径：

    ```bash
    nodes:
      - address: `IP_ADDRESS_1`
        # THE IP ADDRESS OR HOSTNAME OF THE NODE
        user: `USER`
        # USER WITH ADMIN ACCESS. USUALLY `root`
        role: [controlplane,etcd,worker]
        ssh_key_path: ~/.ssh/id_rsa
        # PATH TO SSH KEY THAT AUTHENTICATES ON YOUR WORKSTATION
        # USUALLY THE VALUE ABOVE
      - address: `IP_ADDRESS_2`
        user: `USER`
        role: [controlplane,etcd,worker]
        ssh_key_path: ~/.ssh/id_rsa
      - address: `IP_ADDRESS_3`
        user: `USER`
        role: [controlplane,etcd,worker]
        ssh_key_path: ~/.ssh/id_rsa
    ```

### 2. 证书配置

出于安全考虑，使用Rancher需要SSL加密。 SSL可以保护所有Rancher网络通信，例如登录或与群集交互时。

### 方案A — 使用自签名证书

>**先决条件:**
>
> - 证书必须是`PEM格式`；
> - 证书必须通过`base64`加密；
> - 在您的证书文件中，包含链中的所有中间证书；

1. In `kind: Secret` with `name: cattle-keys-ingress`:

    - Replace `<BASE64_CRT>` with the base64 encoded string of the Certificate file (usually called `cert.pem` or `domain.crt`)
    - Replace `<BASE64_KEY>` with the base64 encoded string of the Certificate Key file (usually called `key.pem` or `domain.key`)

    >**注意:**
    > The base64 encoded string should be on the same line as `tls.crt` or `tls.key`, without any newline at the beginning, in between or at the end.

    **Result:** After replacing the values, the file should look like the example below (the base64 encoded strings should be different):

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

2. In `kind: Secret` with `name: cattle-keys-server`, replace `<BASE64_CA>` with the base64 encoded string of the CA Certificate file (usually called `ca.pem` or `ca.crt`).

    >**Note:**
    > The base64 encoded string should be on the same line as `cacerts.pem`, without any newline at the beginning, in between or at the end.

    **Result:** The file should look like the example below (the base64 encoded string should be different):
    ```yaml
    ---
    apiVersion: v1
    kind: Secret
    metadata:
      name: cattle-keys-server
      namespace: cattle-system
    type: Opaque
    data:
      cacerts.pem:
    ```

### 方案B — 使用权威CA机构颁发的SSL证书

>**Note:**
> If you are using Self Signed Certificate, [click here](#option-a-self-signed-certificate) to proceed.

If you are using a Certificate Signed By A Recognized Certificate Authority, you will need to generate a base64 encoded string for the Certificate file and the Certificate Key file. Make sure that your certificate file includes all the [intermediate certificates](#ssl-faq-troubleshooting) in the chain, the order of certificates in this case is first your own certificate, followed by the intermediates. Please refer to the documentation of your CSP (Certificate Service Provider) to see what intermediate certificate(s) need to be included.

1. In the `kind: Secret` with `name: cattle-keys-ingress`:

    - Replace `<BASE64_CRT>` with the base64 encoded string of the Certificate file (usually called `cert.pem` or `domain.crt`)

    - Replace `<BASE64_KEY>` with the base64 encoded string of the Certificate Key file (usually called `key.pem` or `domain.key`)

    After replacing the values, the file should look like the example below (the base64 encoded strings should be different):

    >**注意:**
    > The base64 encoded string should be on the same line as `tls.crt` or `tls.key`, without any newline at the    beginning, in between or at the end.

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

### 3. 域名配置

There are two references to `<FQDN>` in the config file (one in this step and one in the next). Both need to be replaced with the FQDN chosen in [Configure DNS](#3-configure-dns).

In the `kind: Ingress` with `name: cattle-ingress-http`:

- Replace `<FQDN>` with the FQDN chosen in [Configure DNS](#3-configure-dns).

  After replacing `<FQDN>` with the FQDN chosen in [Configure DNS](#3-configure-dns), the file should look like the example below (`rancher.yourdomain.com` is the FQDN used in this example):

  ```yaml
   ---
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      namespace: cattle-system
      name: cattle-ingress-http
      annotations:
        nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
        nginx.ingress.kubernetes.io/proxy-read-timeout: "1800"   # Max time in seconds for ws to remain shell         window open
        nginx.ingress.kubernetes.io/proxy-send-timeout: "1800"   # Max time in seconds for ws to remain shell         window open
    spec:
      rules:
      - host: rancher.yourdomain.com
        http:
          paths:
          - backend:
              serviceName: cattle-service
              servicePort: 80
      tls:
      - secretName: cattle-keys-ingress
        hosts:
        - rancher.yourdomain.com
  ```

    Save the `.yml` file and close it.

### 4.备份配置文件

After you close your `.yml` file, back it up to a secure location. You can use this file again when it's time to upgrade Rancher. 

## 七、运行RKE

With all configuration in place, use RKE to launch Rancher. You can complete this action by running the `rke up` command and using the `--config` parameter to point toward your config file.

1. From your workstation, make sure `rancher-cluster.yml` and the downloaded `rke` binary are in the same directory.

2. Open a Terminal instance. Change to the directory that contains your config file and `rke`.

3. Enter one of the `rke up` commands listen below.

    ```bash
    # MacOS
    ./rke_darwin-amd64 up --config rancher-cluster.yml
    # Linux
    ./rke_linux-amd64 up --config rancher-cluster.yml
    ```

    **Step Result:** The output should be similar to the snippet below:

    ```bash
    INFO[0000] Building Kubernetes cluster
    INFO[0000] [dialer] Setup tunnel for host [1.1.1.1]
    INFO[0000] [network] Deploying port listener containers
    INFO[0000] [network] Pulling image [alpine:latest] on host [1.1.1.1]
    ...
    INFO[0101] Finished building Kubernetes cluster successfully
    ```

## 八、备份自动生成的kubectl配置文件

在安装过程中，RKE会自动生成一个kube_config_rancher-cluster.yml与RKE二进制文件位于同一目录中的配置文件。此文件很重要，它可以在Rancher server故障时，利用kubectl通过此配置文件管理Kubernetes集群。复制此文件将其备份到安全位置。

## 九、下一步？

You have a couple of options:

- 创建Rancher server的备份：[单节点备份和恢复](/docs/rancher/v2.x/cn/backups-and-restoration/single-node-backup-and-restoration/)；

- 创建一个Kubernetes集群：[创建一个集群](/docs/rancher/v2.x/cn/installation/server-installation/single-node-install/%7B%7B%20%3Cbaseurl%3E%20%7D%7D/rancher/v2.x/en/tasks/clusters/creating-a-cluster/)；

## 十、FAQ and Troubleshooting

1. ### 我如何知道我的证书是否为PEM格式？

    可以通过以下特征识别PEM格式：

    - 该文件以下列标题开头：
    `-----BEGIN CERTIFICATE-----`
    - 标题后面跟着一串长字符
    - 该文件以页脚结尾：
    `-----END CERTIFICATE-----`

    **PEM证书示例: **

    ```bash
    ----BEGIN CERTIFICATE-----
    MIIGVDCCBDygAwIBAgIJAMiIrEm29kRLMA0GCSqGSIb3DQEBCwUAMHkxCzAJBgNV
    ... more lines
    VWQqljhfacYPgp8KJUJENQ9h5hZ2nSCrI+W00Jcw4QcEdCI8HL5wmg==
    -----END CERTIFICATE-----
    ```
2. ### 如何对证书PEM进行base64加密？

    1. 登录终端，切换目录到证书所在目录；

    2. 根据操作系统类型，选择运行以下命令，替换FILENAME为您的证书名称。

    ```bash
    # MacOS
    cat FILENAME | base64
    # Linux
    cat FILENAME | base64 -w0
    # Windows
    certutil -encode FILENAME FILENAME.base64
    ```

3. ### 如何验证base64字符串对应的证书文件？

    1. 复制生成的base64字符串；

    2. 替换`YOUR_BASE64_STRING`,根据操作系统类型，选择运行以下命令:

    ```bash
    # MacOS
    echo YOUR_BASE64_STRING | base64 -D
    # Linux
    echo YOUR_BASE64_STRING | base64 -d
    # Windows
    certutil -decode FILENAME.base64 FILENAME.verify
    ```
4. ### 如果我想添加我的中间证书，证书的顺序是什么？

    添加证书的顺序如下：

    ```bash
    -----BEGIN CERTIFICATE-----
    %YOUR_CERTIFICATE%
    -----END CERTIFICATE-----
    -----BEGIN CERTIFICATE-----
    %YOUR_INTERMEDIATE_CERTIFICATE%
    -----END CERTIFICATE-----
    ```

5. ### 我如何验证我的证书链？

    您可以使用`openssl`二进制验证证书链。如果该命令的输出（参见下面的命令示例）结束`Verify return code: 0 (ok)`，那么证书链是有效的。该`ca.pem`文件必须与您添加到`rancher/rancher`容器中的文件相同。当使用由认可的认证机构签署的证书时，可以省略该`-CAfile`参数。

    **命令:**

    ```bash
    openssl s_client -CAfile ca.pem -connect rancher.yourdomain.com:443
    ...
    Verify return code: 0 (ok)
    ```