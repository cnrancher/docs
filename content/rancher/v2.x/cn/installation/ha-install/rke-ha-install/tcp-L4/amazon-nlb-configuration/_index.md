---
title: Amazon NLB 配置
weight: 1
---

## 一、创建目标组

从技术上讲，只需要端口443来访问Rancher，但是通常我们建议把让80端口也监听，它将自动重定向到端口443。节点上的NGINX控制器将确保端口80被重定向到端口443。

登录[Amazon AWS Console](https://console.aws.amazon.com/ec2/),确定EC2实例(Linux节点)创建的区域。

目标组配置位于EC2服务的负载平衡部分，选择`服务`并选择`EC2`，找到`负载平衡`部分并打开`目标组`:

![EC2 Load Balancing section]({{< baseurl >}}/img/rancher/ha/nlb/ec2-loadbalancing.png)

### 1、目标组(TCP443端口)

单击“ 创建目标组”以创建有关TCP端口443的第一个目标组。根据下表配置第一个目标组，配置截图显示在表格下方。

| 设置选项                              | 设置值           |
| ----------------------------------- | ----------------- |
| Target Group Name                   | `rancher-tcp-443` |
| Protocol                            | `TCP`             |
| Port                                | `443`             |
| Target type                         | `instance`        |
| VPC                                 | 选择您的VPC   |
| Protocol<br/>(Health Check)         | `HTTP`            |
| Path<br/>(Health Check)             | `/healthz`        |
| Port (Advanced health check)        | `override`,`80`   |
| Healthy threshold (Advanced health) | `3`               |
| Unhealthy threshold (Advanced)      | `3`               |
| Timeout (Advanced)                  | `6 seconds`       |
| Interval (Advanced)                 | `10 second`       |
| Success codes                       | `200-399`         |

**Screenshot Target group TCP port 443 settings**
![Target group 443]({{< baseurl >}}/img/rancher/ha/nlb/create-targetgroup-443.png)

**Screenshot Target group TCP port 443 Advanced settings**
![Target group 443 Advanced]({{< baseurl >}}/img/rancher/ha/nlb/create-targetgroup-443-advanced.png)

### 2、目标组(TCP80端口)

单击“ 创建目标组”以创建有关TCP端口80的第二个目标组。

根据下表配置第二个目标组，配置截图显示在表格下方。

| 设置选项                              | 设置值             |
| ----------------------------------- | ----------------   |
| Target Group Name                   | `rancher-tcp-80`   |
| Protocol                            | `TCP`              |
| Port                                | `80`             |
| Target type                         | `instance`       |
| VPC                                 | 选择您的VPC  |
| Protocol<br/>(Health Check)         | `HTTP`           |
| Path<br/>(Health Check)             | `/healthz`       |
| Port (Advanced health check)        | `traffic port`   |
| Healthy threshold (Advanced health) | `3`              |
| Unhealthy threshold (Advanced)      | `3`              |
| Timeout (Advanced)                  | `6 seconds`      |
| Interval (Advanced)                 | `10 second`      |
| Success codes                       | `200-399`        |

**Screenshot Target group TCP port 80 settings**
![Target group 80]({{< baseurl >}}/img/rancher/ha/nlb/create-targetgroup-80.png)

**Screenshot Target group TCP port 80 Advanced settings**
![Target group 80 Advanced]({{< baseurl >}}/img/rancher/ha/nlb/create-targetgroup-80-advanced.png)

## 二、注册目标

接下来，将Linux节点添加到两个目标组。选择名为`rancher-tcp-443`的目标组，单击选项卡`Targets`并选择`Edit`。

![Edit target group 443]({{< baseurl >}}/img/rancher/ha/nlb/edit-targetgroup-443.png)

选择要添加的实例(Linux节点)，然后单击“ 添加到注册”。

**Screenshot Add targets to target group TCP port 443**
![Add targets to target group 443]({{< baseurl >}}/img/rancher/ha/nlb/add-targets-targetgroup-443.png)

**Screenshot Added targets to target group TCP port 443**
![Added targets to target group 443]({{< baseurl >}}/img/rancher/ha/nlb/added-targets-targetgroup-443.png)

添加实例后，单击屏幕右下角的“ 保存 ”。重复这些步骤，注册rancher-TCP-80。

## 三、创建NLB

1、浏览器访问[Amazon EC2 Console](https://console.aws.amazon.com/ec2/)；

2、从导航窗格中，选择`LOAD BALANCING`> `Load Balancers`；

3、点击 **Create Load Balancer**；

4、选择 **Network Load Balancer** 并点击**Create**；

5、**配置负载均衡配置**:

- Name: `rancher`
- Scheme: `internet-facing`
- **监听**: 在下面添加负载均衡器协议和负载均衡器端口。
  - `TCP`: `443`
- **可用区域**
  - 选择您的VPC和可用区

6、**配置路由表**.

- 从“ 目标组”下拉列表中，选择“ 现有目标组”
- 从“ 名称”下拉列表中选择rancher-tcp-443。
- 打开高级运行状况检查设置，并将Interval配置为10 seconds;

7、**注册目标**.

因为之前注册了目标，所以只需单击`下一步:查看`;

8、**验证**. 查看负载均衡器详细信息，确认无误后单击`创建`。

9、AWS创建NLB后，单击“ 关闭”。

## 四、添加TCP80端口到NLB监听

1. 选择新创建的NLB并选择`Listeners`选项;

2. 点击 **Add listener**；

3. **Protocol**和**Port**分别选择`TCP`:`80`；

4. 点击**Add action**并选择**Forward to...**；

5. 通过**Forward to**的下拉列表, 选择`rancher-tcp-80`；

6. 单击屏幕右上角的“ 保存 ”。