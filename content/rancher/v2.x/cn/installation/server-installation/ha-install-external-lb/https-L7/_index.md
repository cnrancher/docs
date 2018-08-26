---
title: 2 - 七层负载均衡HA部署
weight: 2
---

以下步骤将创建一个新的Kubernetes集群，专用于Rancher server高可用(HA)运行,本文档将引导你使用Rancher Kubernetes Engine(RKE)配置三个节点的集群.

## 一、架构说明

![Rancher HA]({{< baseurl >}}/img/rancher/ha/rancher2ha-l7.svg)

## 二、Linux主机要求

在安装Rancher之前，请确认符合主机要求,使用以下要求配置3个新的Linux主机。

### 1、操作系统

- Ubuntu 16.04(64位)
- Centos/RedHat Linux 7.5+(64位)
- RancherOS 1.3.0+(64位)

### 2、硬件

硬件需求根据Rancher部署的规模进行扩展。根据需求配置每个节点。

| 部署大小 | 集群(个)  | 节点(个) | vCPU                                            | 内存 |
| -------- | --------- | -------- |     ------------------------------------------- | ---- |
| 小       | 不超过10  | 最多50   | 2C                                              | 4GB  |
| 中       | 不超过100 | 最多500  | 8C                                              | 32GB |
| 大       | 超过100   | 超过500  | [联系Rancher](https://www.cnrancher.com/contact/) |      |

### 3、软件

- Docker

    >**注意:**如果你使用的是RancherOS，请确保你将Docker引擎切换为受支持的版本`sudo ros engine switch docker-17.03.2-ce`

    **支持的Docker版本**

  - `1.12.6`
  - `1.13.1`
  - `17.03.2`

    [Docker安装说明](https://docs.docker.com/install/)

    >**注意:**1.该`rancher/rancher`镜像托管在[DockerHub上](https://hub.docker.com/r/rancher/rancher/tags/)。如果你无法访问DockerHub，或者离线环境下安装Rancher，请查阅[离线安装](/docs/rancher/v2.x/cn/installation/server-installation/air-gap-installation/)。\
    >2.更多Rancher server tag列表，请查阅[Rancher server tags](/docs/rancher/v2.x/cn/installation/server-tags/)。

### 4、端口

下图描述了Rancher的基本端口要求。有关全面列表，请查阅[端口要求](/docs/rancher/v2.x/cn/installation/references/)。

![基本端口要求](/docs/img/rancher/port-communications.png)

## 三、配置负载均衡器(以NGINX为例)

默认情况下，通过`docker run`运行的Rancher server容器会自动把端口80重定向到443，但是通过负载均衡器来代理Rancher server容器后，不再需要将Rancher server容器端口从80重定向到443。通过在负载均衡器上配置`X-Forwarded-Proto: https`参数后，Rancher server容器端口重定向功能将自动被禁用。

负载均衡器或代理必须支持以下参数:

- **WebSocket** 连接
- **SPDY**/**HTTP/2**协议
- 传递/设置以下headers:

| Header              | Value             | 描述            |
|---------------------|--------------------------|:------------|
| `Host`              | 传递给Rancher的主机名| 识别客户端请求的主机名。      |
| `X-Forwarded-Proto` | `https`       | 识别客户端用于连接负载均衡器的协议。**注意：**如果存在此标头，`rancher / rancher`不会将HTTP重定向到HTTPS。 |
| `X-Forwarded-Port`  | Port used to reach Rancher.   | 识别客户端用于连接负载均衡器的协议。      |
| `X-Forwarded-For`   | IP of the client connection.   | 识别客户端的原始IP地址。            |

我们有以下负载均衡器的示例配置:

- [Amazon ALB](./alb)
- [nginx](./nginx)

## 四、配置DNS

选择一个用于访问Rancher的域名(FQDN)(例如，demo.rancher.com).

### 1、方案1 - 有DNS服务器

- 1、登录DNS服务，创建一条 `A` 记录指向负载均衡主机IP；

- 2、在终端中执行一下命令来验证运行解析是否生效:

    `nslookup HOSTNAME.DOMAIN.COM`

    **如果解析生效**

    ```bash
    nslookup demo.rancher.com
    DNS Server:         YOUR_HOSTNAME_IP_ADDRESS
    DNS Address:        YOUR_HOSTNAME_IP_ADDRESS#53
    Non-authoritative answer:
    Name:   demo.rancher.com
    Address: <负载均衡IP地址>
    ```
    **如果解析不生效**

    ```bash
    nslookup demo.rancher.com
    DNS Server:         YOUR_HOSTNAME_IP_ADDRESS
    DNS Address:        YOUR_HOSTNAME_IP_ADDRESS#5
    ** server can't find demo.rancher.com: NXDOMAIN
    ```

### 2、方案2 - 无DNS服务器

如果环境为内部网络且无DNS服务器，可以通过修改客户端的`/etc/hosts`文件，添加相应的条目。比如:

![image-20180711140926370](/docs/img/rancher/ha/image-20180711140926370.png)

## 五、下载 RKE

RKE是一种快速，通用的Kubernetes安装程序，可用于在Linux主机上安装Kubernetes。我们将使用RKE来配置Kubernetes集群并运行Rancher。

1、打开浏览器访问[下载文件](/docs/rancher/v2.x/cn/installation/download/)页面，根据你操作系统类型下载最新版本的RKE:

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

**步骤结果:** 你将看到以下内容:

```bash
rke version v<N.N.N>
```

## 六、下载RKE配置模板

RKE通过 `.yml` 配置文件来安装和配置Kubernetes集群，有2个模板可供选择，具体取决于使用的SSL证书类型。

### 1、根据你使用的SSL证书类型，选择模板下载:

- [Template for self-signed certificate<br/> `3-node-externalssl-certificate.yml`](https://raw.githubusercontent.com/rancher/rancher/58e695b51096b1f404188379cea6f6a35aea9e4c/rke-templates/3-node-externalssl-certificate.yml)

- [Template for certificate signed by recognized CA<br/> `3-node-externalssl-recognizedca.yml`](https://raw.githubusercontent.com/rancher/rancher/58e695b51096b1f404188379cea6f6a35aea9e4c/rke-templates/3-node-externalssl-recognizedca.yml)

### 2、重命名模板文件为 `rancher-cluster.yml`.

## 七、节点配置

获得rancher-cluster.yml配置文件模板后，编辑节点部分以指向Linux主机。

### 1、节点免密登录

- 第一步:在任意一台Linux主机使用ssh-keygen命令产生公钥私钥对

    ```bash
    ssh-keygen
    ```

- 第二步:通过ssh-copy-id命令将公钥复制到远程机器中

    ```bash
    ssh-copy-id -i .ssh/id_rsa.pub  $user@192.168.x.xxx
    ```

### 2、编辑`rancher-cluster.yml`配置文件

编辑器打开 `rancher-cluster.yml` 文件,在nodes配置版块中，修改 `IP_ADDRESS_X` and `USER`为你真实的Linux主机IP和用户名,`ssh_key_path`为第一步生成的私钥文件，如果是在RKE所在主机上生成的公钥私钥对，此配置可保持默认:

```yaml
nodes:
  - address: IP_ADDRESS_1
    # THE IP ADDRESS OR HOSTNAME OF THE NODE
    user: USER
    # USER WITH ADMIN ACCESS. USUALLY `root`
    role: [controlplane,etcd,worker]
    ssh_key_path: ~/.ssh/id_rsa
    # PATH TO SSH KEY THAT AUTHENTICATES ON YOUR WORKSTATION
    # USUALLY THE VALUE ABOVE
  - address: IP_ADDRESS_2
    user: USER
    role: [controlplane,etcd,worker]
    ssh_key_path: ~/.ssh/id_rsa
  - address: IP_ADDRESS_3
    user: USER
    role: [controlplane,etcd,worker]
    ssh_key_path: ~/.ssh/id_rsa
```

> **注意** 使用RHEL/CentOS系统时，因为系统安全限制，`ssh`不能使用root账户。

## 八、证书配置

出于安全考虑，使用Rancher需要SSL加密。 SSL可以保护所有Rancher网络通信，例如登录或与集群交互时。

### 1、方案A — 使用自签名证书

>**先决条件:** 1.证书必须是`PEM格式`,`PEM`只是一种证书类型，并不是说文件必须是PEM为后缀，具体可以查看[证书类型](/docs/rancher/v2.x/cn/installation/self-signed-ssl/)；\
> 2.证书必须通过`base64`加密；\
> 3.在你的证书文件中，包含链中的所有中间证书；

在`kind: Secret`和`name: cattle-keys-ingress`中:

- 替换<BASE64_CA>为CA证书文件的base64编码字符串(通常称为ca.pem或ca.crt)

>**注意:** base64编码的字符串应该与cacerts.pem在同一行，冒号后有一个空格，在开头，中间或结尾没有任何换行符。

结果:替换值后，文件应如下所示(base64编码的字符串应该不同):

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: cattle-keys-server
  namespace: cattle-system
type: Opaque
data:
    cacerts.pem: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNvRENDQVlnQ0NRRHVVWjZuMEZWeU16QU5CZ2txaGtpRzl3MEJBUXNGQURBU01SQXdEZ1lEVlFRRERBZDAKWlhOMExXTmhNQjRYRFRFNE1EVXdOakl4TURRd09Wb1hEVEU0TURjd05USXhNRFF3T1Zvd0VqRVFNQTRHQTFVRQpBd3dIZEdWemRDMWpZVENDQVNJd0RRWUpLb1pJaHZjTkFRRUJCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFNQmpBS3dQCndhRUhwQTdaRW1iWWczaTNYNlppVmtGZFJGckJlTmFYTHFPL2R0RUdmWktqYUF0Wm45R1VsckQxZUlUS3UzVHgKOWlGVlV4Mmo1Z0tyWmpwWitCUnFiZ1BNbk5hS1hocmRTdDRtUUN0VFFZdGRYMVFZS0pUbWF5NU45N3FoNTZtWQprMllKRkpOWVhHWlJabkdMUXJQNk04VHZramF0ZnZOdmJ0WmtkY2orYlY3aWhXanp2d2theHRUVjZlUGxuM2p5CnJUeXBBTDliYnlVcHlad3E2MWQvb0Q4VUtwZ2lZM1dOWmN1YnNvSjhxWlRsTnN6UjVadEFJV0tjSE5ZbE93d2oKaG41RE1tSFpwZ0ZGNW14TU52akxPRUc0S0ZRU3laYlV2QzlZRUhLZTUxbGVxa1lmQmtBZWpPY002TnlWQUh1dApuay9DMHpXcGdENkIwbkVDQXdFQUFUQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFHTCtaNkRzK2R4WTZsU2VBClZHSkMvdzE1bHJ2ZXdia1YxN3hvcmlyNEMxVURJSXB6YXdCdFJRSGdSWXVtblVqOGo4T0hFWUFDUEthR3BTVUsKRDVuVWdzV0pMUUV0TDA2eTh6M3A0MDBrSlZFZW9xZlVnYjQrK1JLRVJrWmowWXR3NEN0WHhwOVMzVkd4NmNOQQozZVlqRnRQd2hoYWVEQmdma1hXQWtISXFDcEsrN3RYem9pRGpXbi8walI2VDcrSGlaNEZjZ1AzYnd3K3NjUDIyCjlDQVZ1ZFg4TWpEQ1hTcll0Y0ZINllBanlCSTJjbDhoSkJqa2E3aERpVC9DaFlEZlFFVFZDM3crQjBDYjF1NWcKdE03Z2NGcUw4OVdhMnp5UzdNdXk5bEthUDBvTXl1Ty82Tm1wNjNsVnRHeEZKSFh4WTN6M0lycGxlbTNZQThpTwpmbmlYZXc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
```

### 2、方案B—使用权威CA机构颁发的证书

如果你使用的是权威CA机构颁发的证书，则不需要做任何操作。

## 九、域名配置

RKE配置文件中有一个`<FQDN>`引用,编辑配置文件替换`<FQDN>`:

**结果**: 替换值后，文件应如下所示(base64编码的字符串应该不同):

```yaml
apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    namespace: cattle-system
    name: cattle-ingress-http
    annotations:
      nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
      nginx.ingress.kubernetes.io/proxy-read-timeout: "1800"   # Max time in seconds for ws toremain shell window open
      nginx.ingress.kubernetes.io/proxy-send-timeout: "1800"   # Max time in seconds for ws toremain shell window open
  spec:
    rules:
    - host: demo.rancher.com
      http:
        paths:
        - backend:
            serviceName: cattle-service
            servicePort: 80
```

## 十、备份配置文件

保存关闭.yml文件后，将其备份到安全位置。升级Rancher时，你需要再次使用此文件。

## 十一、运行RKE

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

```bash
INFO[0000] Building Kubernetes cluster
INFO[0000] [dialer] Setup tunnel for host [1.1.1.1]
INFO[0000] [network] Deploying port listener containers
INFO[0000] [network] Pulling image [alpine:latest] on host [1.1.1.1]
...
INFO[0101] Finished building Kubernetes cluster successfully
```

## 十二、备份自动生成的kubectl配置文件

在安装过程中，RKE会自动生成一个kube_config_rancher-cluster.yml与RKE二进制文件位于同一目录中的配置文件。此文件很重要，它可以在Rancher server故障时，利用kubectl通过此配置文件管理Kubernetes集群。复制此文件将其备份到安全位置。

## 十三、下一步？

你有几个选择:

- 创建Rancher server的备份:[单节点备份和恢复](/docs/rancher/v2.x/cn/backups-and-restoration/backups/single-node-backups/)。
- 创建一个Kubernetes集群:[创建一个集群](/docs/rancher/v2.x/cn/configuration/clusters/creating-a-cluster/)。

## 十四、FAQ and Troubleshooting

### 1、我如何知道我的证书是否为PEM格式？

可以通过以下特征识别PEM格式:

```bash
该文件以下列标题开头:
-----BEGIN CERTIFICATE-----
标题后面跟着一串长字符
该文件以页脚结尾:
-----END CERTIFICATE-----
```

**PEM证书示例:**

```bash
----BEGIN CERTIFICATE-----
MIIGVDCCBDygAwIBAgIJAMiIrEm29kRLMA0GCSqGSIb3DQEBCwUAMHkxCzAJBgNV
... more lines
VWQqljhfacYPgp8KJUJENQ9h5hZ2nSCrI+W00Jcw4QcEdCI8HL5wmg==
-----END CERTIFICATE-----
```

### 2、如何对证书PEM进行base64加密？

- 登录终端，切换目录到证书所在目录；

- 根据操作系统类型，选择运行以下命令，替换FILENAME为你的证书名称。

    ```bash
    # MacOS
    cat FILENAME | base64
    # Linux
    cat FILENAME | base64 -w0
    # Windows
    certutil -encode FILENAME FILENAME.base64
    ```

### 3、如何验证base64字符串对应的证书文件？

- 复制生成的base64字符串；

- 替换`YOUR_BASE64_STRING`,根据操作系统类型，选择运行以下命令:

    ```bash
    # MacOS
    echo YOUR_BASE64_STRING | base64 -D
    # Linux
    echo YOUR_BASE64_STRING | base64 -d
    # Windows
    certutil -decode FILENAME.base64 FILENAME.verify
    ```

### 4、如果我想添加我的中间证书，证书的顺序是什么？

添加证书的顺序如下:

```bash
-----BEGIN CERTIFICATE-----
%YOUR_CERTIFICATE%
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
%YOUR_INTERMEDIATE_CERTIFICATE%
-----END CERTIFICATE-----
```

### 5、如何验证我的证书链？

你可以使用`openssl`二进制验证证书链。如果该命令的输出(参见下面的命令示例)结束`Verify return code: 0 (ok)`，那么证书链是有效的。该`ca.pem`文件必须与你添加到`rancher/rancher`容器中的文件相同。当使用由认可的认证机构签署的证书时，可以省略该`-CAfile`参数。

**命令:**

```bash
openssl s_client -CAfile ca.pem -connect demo.rancher.com:443
...
Verify return code: 0 (ok)
```