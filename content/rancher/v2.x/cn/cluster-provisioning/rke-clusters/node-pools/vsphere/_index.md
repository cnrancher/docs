---
title: 4 - vSphere
shortTitle: vSphere
weight: 4
---

使用Rancher通过主机驱动在vSphere中创建Kubernetes集群。

## 一、介绍

创建vSphere集群时，Rancher首先通过与vCenter API通信来配置指定数量的虚拟机，然后在虚拟机之上安装Kubernetes。vSphere集群可能包含多组具有不同属性的VM，例如内存或vCPU。该分组允许分别对`ETCD`，`Control`和`worker`节点大小进行细粒度控制。

>**注意:**
>Rancher中包含的vSphere节点驱动程序目前仅支持使用`RancherOS`作为客户机操作系统来配置VM 。

## 二、先决条件

在继续创建集群之前，必须确保具有足够权限的vSphere用户。如果您计划将vSphere存储卷用于集群中的持久存储，则还必须满足[其他要求]({{< baseurl >}}/rke/v0.1.x/en/config-options/cloud-providers/vsphere/)。

## 三、配置用户权限

通过以下步骤创建所需权限的角色，然后在vSphere控制台中将其分配给新用户：

1. 在`vSphere控制台`中，转至`管理`页面。

2. 转到`角色`选项卡。

3. 创建一个新角色。为其命名并选择[权限表](#附件-vsphere权限)中列出的权限。

    ![image]({{< baseurl >}}/img/rancher/rancherroles1.png)

4. 转到`用户和组`选项卡。

5. 创建一个新用户。填写表单，然后单击`确定`。确保记下用户名和密码，因为在Rancher中配置节点模板时将需要设置它。

    ![image]({{< baseurl >}}/img/rancher/rancheruser.png)

6. 转到`全局权限`选项卡。

7. 创建新的全局权限。添加您之前创建的用户，并为其分配您之前创建的角色，最后单击`确定`。

    ![image]({{< baseurl >}}/img/rancher/globalpermissionuser.png)

    ![image]({{< baseurl >}}/img/rancher/globalpermissionrole.png)

## 四、创建主机模板

要创建集群，需要创建至少一个vSphere[主机模板]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/rke-clusters/node-pools/#node-templates)，该模板指定如何创建VM。

>**注意:** 创建节点模板后，将保存该模板，并且可以在创建其他vSphere集群时重复使用它。

1. 使用管理员帐户登录Rancher UI。

2. 从用户设置菜单中，选择`主机模板`。

3. 单击`添加模板`，然后单击`vSphere`图标。

4. 在[帐户访问](#帐户访问权限)中输入vCenter FQDN或IP地址以及vSphere用户帐户的凭据(查看[先决条件](#二-先决条件))。

5. 在[实例选项](#实例选项)中, 配置此模板创建的VM的vCPU数、内存和磁盘大小。

6. **可选:** 在[Cloud Init](#实例选项)中输入[RancherOS]({{< baseurl >}}/os/v1.x/en/) cloud-config文件的URL地址。

7. 确保[OS ISO URL](#实例选项)包含RancherOS(`rancheros-vmware.iso`)的VMware ISO版本的URL。

    >**注意:** 如果是离线环境，此URL可以设置为一个内部的`http URL`地址，默认是公网地址。

    ![image]({{< baseurl >}}/img/rancher/vsphere-node-template-1.png)

8. **可选:** 为虚拟机设置一组[配置参数](#实例选项)。

9. 在**调度**中, 输入要创建VM的数据中心的`名称`，虚拟机要使用的网络名称，以及用于存储磁盘的数据存储`名称`。
    >资源池（pool）书写格式：`/<dataCenter>/host/<ClusterName>/Resources/<poolName>`, `< >`表示需要修改的参数，其他为固定格式。
    >主机（host）书写格式：`<ClusterName>/<host_name>`, `< >`表示需要修改的参数。

    ![image]({{< baseurl >}}/img/rancher/vsphere-node-template-2.png)

10. 为这个模板指定一个描述性的**名称**。

11. **可选:** 为POD的调度设置主机标签。

12. **可选:** 在将要创建的vm上定制Docker守护进程的配置。

    常见的配置有Docker存储驱动、私有仓库地址、加速镜像地址

    ![image-20181110214427204](_index.assets/image-20181110214427204.png)

13. 最后点击`创建`。

## 五、创建vSphere Cluster

创建模板后，可以使用它来创建vSphere虚拟机和K8S集群。

1. 在`全局`视图中，单击`添加集群`。

2. 选择`vSphere`图表。

3. 设置`Cluster`名称。

4. 使用`成员角色`配置集群的用户授权。

    - 单击`添加成员`以添加可以访问集群的用户。
    - 使用`角色`下拉列表为每个用户设置权限。

5. 根据实际需求配置`集群选项`。

6. 将一个或多个节点池添加到集群。

    ![image]({{< baseurl >}}/img/rancher/vsphere-cluster-create-1.png)

7. 检查配置，然后单击`创建`。

> **注意:** 如果您的集群启用了`DRS`, 建议设置[VM-VM关联规则](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.resmgmt.doc/GUID-7297C302-378F-4AF2-9BD6-6EDB1E0A850A.html)。这些规则允许分配了`etcd`和`Control`角色的VM在分配给不同节点池时在不同的ESXi主机上运行,这种做法可确保单个物理机器的故障不会影响这些平面的可用性。

## 附件-节点模板配置参考

下表介绍了vSphere主机模板中可用的配置选项。

### 帐户访问权限

| 参数                         | 需要 | 描述 |
|:------------------------|:--------|:------------------------------------------------------------|
| vCenter or ESXi Server   |   *      | 用于管理VM的vCenter或ESXi服务器的IP或FQDN |
| Port                     |   *      | 连接到服务器时使用的端口，默认为`443`  |
| Username                 |   *      | vCenter/ESXi用户要对服务器进行身份验证 |
| Password                 |   *      | 用户密码|

### 实例选项

|   参数                    |  需要  | 描述 |
|:-------------------------|:---------|:--------------------------------------------------------------|
| CPUs                     |   *      | 分配给VM的vCPUS数 |
| Memory                   |   *      | 分配给VM的内存大小 |
| Disk                     |   *      | 要附加到VM的磁盘大小（以MB为单位） |
| Cloud Init               |          | 用于配置VM的[RancherOS cloud-config]({{< baseurl >}}/os/v1.x/en/installation/configuration/)文件的URL 。此文件允许进一步自定义RancherOS操作系统，例如网络配置，DNS服务器或系统守护程序。|
| OS ISO URL               |   *      | 用于引导VM的RancherOS vSphere ISO文件的URL，可以在 [Rancher OS GitHub Repo](https://github.com/rancher/os)中找到特定版本的URL 。 |
| Configuration Parameters |          | VM的其他配置参数。这些对应于vSphere控制台中[Advanced Settings](https://kb.vmware.com/s/article/1016098) ，示例用例包括提供RancherOS [guestinfo]({{< baseurl >}}/os/v1.x/en/installation/running-rancheros/cloud/vmware-esxi/#vmware-guestinfo)参数或为VM（`disk.EnableUUID=TRUE`）启用磁盘UUID 。 |

### 调度选项

| 参数                    | 需要 | 描述 |
|:------------------------|:--------|:------------------------------------------------------------|
| Data Center              |   *      | 用于创建VM的数据中心的`名称/路`       |
| Pool                     |          | 用于调度VM的资源池的`名称/路径`,如果未指定，则使用默认资源池。 |
| Host                     |          | 用于调度VM的主机的`名称/路径`, 如果指定，将使用主机的pool，并忽略Pool参数。 |
| Network                  |   *      | VM需要使用的`网络名称` |
| Data Store               |   *      | 用于分配VM磁盘的`数据存储` |
| Folder                   |          | 数据存储中用于创建VM的文件夹的`名称/路径`。如果要配置，则路径必须已存在。 |

## 附件- vSphere权限

下表列出了节点模板中配置的vSphere用户帐户所需的权限：

| 权限组     | 操作  |
|:----------------------|:-----------------------------------------------------------------------|
| Datastore             | AllocateSpace </br> Browse </br> FileManagement </br> UpdateVirtualMachineFiles </br> UpdateVirtualMachineMetadata |
| Network               | Assign |
| Resource              | AssignVMToPool |
| Virtual Machine       | Config (All) </br> GuestOperations (All) </br> Interact (All) </br> Inventory (All) </br> Provisioning (All) |
