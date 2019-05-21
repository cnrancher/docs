---
title: 2 - 七层负载均衡HA部署
weight: 2
---

>### **重要提示:**
>RKE HA安装仅支持Rancher v2.0.8以及之前的版本，Rancher v2.0.8之后的版本使用[helm安装Rancher]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/)。

以下步骤将创建一个新的Kubernetes集群，专用于Rancher server高可用(HA)运行,本文档将引导你使用Rancher Kubernetes Engine(RKE)配置三个节点的集群.

## 一、架构说明

![Rancher HA]({{< baseurl >}}/img/rancher/ha/rancher2ha-l7.svg)

## 一、Linux主机要求

- [基础环境配置]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/basic-environment-configuration/)
- [端口需求]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/references/)

## 二、配置负载均衡器(以NGINX为例)

默认情况下，rancher容器会将80端口上的请求重定向到443端口上。如果rancher server通过负载均衡器来代理，这个时候请求是通过负载均衡器发送给rancher server，而并非客户端直接访问rancher server。在非全局`https`的环境中，如果以外部负载均衡器作为ssl终止，这个时候通过负载均衡器的`https`请求将需要被反向代理到rancher server http(80)上。在负载均衡器上配置`X-Forwarded-Proto: https`参数，rancher server http(80)上收到负载均衡器的请求后，就不会再重定向到https(443)上。

负载均衡器或代理必须支持以下参数:

- **WebSocket** 连接
- **SPDY**/**HTTP/2**协议
- 传递/设置以下headers:

| Header              | Value             | 描述            |
|---------------------|--------------------------|:------------|
| `Host`              | 传递给Rancher的主机名| 识别客户端请求的主机名。      |
| `X-Forwarded-Proto` | `https`       | 识别客户端用于连接负载均衡器的协议。**注意：**如果存在此标头，`rancher/rancher`不会将HTTP重定向到HTTPS。 |
| `X-Forwarded-Port`  | Port used to reach Rancher.   | 识别客户端用于连接负载均衡器的端口。      |
| `X-Forwarded-For`   | IP of the client connection.   | 识别客户端的原始IP地址。            |

我们有以下负载均衡器的示例配置:

- [Amazon ALB](./alb)
- [nginx](./nginx)

## 三、配置DNS

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

![image-20180711140926370]({{< baseurl >}}/img/rancher/ha/image-20180711140926370.png)

## 四、下载 RKE

RKE是一种快速，通用的Kubernetes安装程序，可用于在Linux主机上安装Kubernetes。我们将使用RKE来配置Kubernetes集群并运行Rancher。

1、打开浏览器访问[下载文件]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/download/)页面，根据你操作系统类型下载最新版本的RKE:

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

## 五、下载RKE配置模板

RKE通过 `.yml` 配置文件来安装和配置Kubernetes集群，有2个模板可供选择，具体取决于使用的SSL证书类型。

1. 根据你使用的SSL证书类型，选择模板下载:

    - [Template for self-signed certificate<br/> `3-node-externalssl-certificate.yml`](https://raw.githubusercontent.com/rancher/rancher/58e695b51096b1f404188379cea6f6a35aea9e4c/rke-templates/3-node-externalssl-certificate.yml)

    - [Template for certificate signed by recognized CA<br/> `3-node-externalssl-recognizedca.yml`](https://raw.githubusercontent.com/rancher/rancher/58e695b51096b1f404188379cea6f6a35aea9e4c/rke-templates/3-node-externalssl-recognizedca.yml)

2. 重命名模板文件为 `rancher-cluster.yml`

## 六、节点配置

获得rancher-cluster.yml配置文件模板后，编辑节点部分以指向Linux主机。

1. 节点免密登录

    - 第一步:在任意一台Linux主机使用ssh-keygen命令产生公钥私钥对

    ```bash
    ssh-keygen
    ```

    - 第二步:通过ssh-copy-id命令将公钥复制到远程机器中

    ```bash
    ssh-copy-id -i .ssh/id_rsa.pub  $user@192.168.x.xxx
    ```

2. 编辑`rancher-cluster.yml`配置文件

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

    >**注意**
    >1、使用RHEL/CentOS系统时，因为系统安全限制，`ssh`不能使用root账户。\
    >2、需要开启[API审计日志？]({{< baseurl >}}/rancher/v2.x/cn/configuration/admin-settings/api-auditing/)\
    >3、了解RKE[配置参数]({{< baseurl >}}/rke/latest/cn/config-options/)

## 七、证书配置

出于安全考虑，使用Rancher需要SSL加密。 SSL可以保护所有Rancher网络通信，例如登录或与集群交互时。

> 如果没有自己的ssl证书，可访问[自签名ssl证书]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/self-signed-ssl/)一键生成自签名ssl证书。

1. 方案A — 使用自签名证书

    >**先决条件:** 1.证书必须是`PEM格式`,`PEM`只是一种证书类型，并不是说文件必须是PEM为后缀，具体可以查看[证书类型]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/self-signed-ssl/)；\
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

2. 方案B—使用权威CA机构颁发的证书

    如果你使用的是权威CA机构颁发的证书，则不需要做任何操作。

## 八、域名配置

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

## 九、备份配置文件

保存关闭.yml文件后，将其备份到安全位置。升级Rancher时，你需要再次使用此文件。

## 十、运行RKE

完成所有配置后，你可以通过运行rke up命令并使用--config参数指定配置文件来完成Rancher 集群的安装。

1. 下载RKE二进制文档到你的主机，确保 `rancher-cluster.yml`与下载的`rke` 在同一目录下；

2. 打开shell 终端，切换路径到RKE所在的目录；

3. 根据操作系统类型，选择以下命令并执行:

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

## 十一、备份自动生成的kubectl配置文件

在安装过程中，RKE会自动生成一个kube_config_rancher-cluster.yml与RKE二进制文件位于同一目录中的配置文件。此文件很重要，它可以在Rancher server故障时，利用kubectl通过此配置文件管理Kubernetes集群。复制此文件将其备份到安全位置。

## 十二、(可选)为Agent Pod添加主机别名(/etc/hosts)

如果你没有内部DNS服务器而是通过添加`/etc/hosts`主机别名的方式指定的Rancher server域名，那么不管通过哪种方式(自定义、导入、Host驱动等)创建K8S集群，K8S集群运行起来之后，因为`cattle-cluster-agent Pod`和`cattle-node-agent`无法通过DNS记录找到`Rancher server`,最终导致无法通信。

### 解决方法

可以通过给`cattle-cluster-agent Pod`和`cattle-node-agent`添加主机别名(/etc/hosts)，让其可以正常通信`(前提是IP地址可以互通)`。

1. cattle-cluster-agent pod

    ```bash
    export kubeconfig=xxx/xxx/xx.kubeconfig.yaml

    kubectl --kubeconfig=$kubeconfig -n cattle-system \
    patch deployments cattle-cluster-agent --patch '{
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
    export kubeconfig=xxx/xxx/xx.kubeconfig.yaml

    kubectl --kubeconfig=$kubeconfig -n cattle-system \
    patch  daemonsets cattle-node-agent --patch '{
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
