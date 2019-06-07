---
title: Amazon ALB配置
weight: 2
---

## 一、创建目标组

登录[Amazon AWS Console](https://console.aws.amazon.com/ec2/)开始配置步骤，具体可以信息可以查询[Amazon Documentation: Create a Target Group](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-target-group.html)

### 1、目标组(HTTP)

设置选项                         | 设置值
--------------------------------|------------------------------------
Target Group Name               | `rancher-http-80`
Protocol                        | `HTTP`
Port                           | `80`
Target type                     | `instance`
VPC                             | 选择您的VPC
Protocol (Health Check)     | `HTTP`
Path (Health Check)         | `/healthz`

### 2、注册目标

接下来，将Linux节点添加到目标组。[Amazon Documentation: Register Targets with Your Target Group](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-register-targets.html)

1、浏览器访问[Amazon EC2 Console](https://console.aws.amazon.com/ec2/).

2、从导航窗格中，选择`LOAD BALANCING` > `Load Balancers`；

3、点击**创建负载均衡**；

4、选择**应用负载均衡**；

5、**步骤 1:配置Load Balancer**:

基础配置

- Name: `rancher-http`
- Scheme: `internet-facing`
- IP address type: `ipv4`

**监听**:添加**负载协议**和**负载端口**

- `HTTP`: `80`
- `HTTPS`: `443`

**可用区域**

- 选择您的VPC和可用区

6、**步骤 2: 配置安全设置**

配置要用于SSL的证书。

7、**步骤 3: 配置安全组**

8、**步骤 4: 配置路由表**:

- 点击`目标组`下拉菜单，选择现有`目标组`;

- 添加目标主机`rancher-http-80`;

9、**步骤 5: 注册目标**

由于您之前已经注册了目标，所以您只需单击下一步:**审计**；

10、**步骤 6: 审计**

确认负载均衡详细信息，确认无误后点击`创建`

11、等ALB创建完成，最后点击**关闭**
