---
title: 2 - 网络插件
weight: 2
---

RKE提供了以下网络插件，这些插件被部署为附加组件:

- Flannel
- Calico
- Canal
- Weave

默认情况下，网络插件是`canal`。如果您想使用另一个网络插件，您需要在`cluster.yml`中指定要在集群级别启用的网络插件类型。

```yaml
# Setting the flannel network plug-in
network:
    plugin: flannel
```

用于网络插件的镜像位于[`system_images` directive]({{< baseurl >}}/rke/latest/cn/config-options/system-images/)。对于每个Kubernetes版本，都有与每个网络插件相关联的默认镜像，但是可以通过更改`system_images`中的镜像版本来覆盖默认版本。

## 一、禁用网络插件的部署

您可以在集群配置中设置`network.plugin为none`来禁止网络插件部署。

```yaml
network:
    plugin: none
```

## 二、网络插件选项

除了可以用于部署网络插件的不同镜像之外，某些网络插件还支持自定义网络插件选项。

### 1、Canal 网络插件选项

```yaml
network:
    plugin: canal
    options:
        canal_iface: eth1
        canal_flannel_backend_type: vxlan
```

#### Canal 接口

通过设置`canal_iface`，您可以配置用于主机间通信接口。`canal_flannel_backend_type`选项允许您指定要使用的[flannel后端](https://github.com/coreos/flannel/blob/master/Documentation/backends.md)类型，默认情况下使用`vxlan`后端。

### 2、Flannel 网络插件选项

```yaml
network:
    plugin: flannel
    options:
        flannel_iface: eth1
        flannel_backend_type: vxlan
```

#### Flannel 接口

通过设置`flannel_iface`，您可以设置用于主机间通信的接口。`flannel_backend_type`选项允许您指定要使用的[flannel后端](https://github.com/coreos/flannel/blob/master/Documentation/backends.md)类型，默认情况下使用`vxlan`后端。

### 3、Calico 网络插件选项

```yaml
network:
    plugin: calico
    options:
        calico_cloud_provider: aws
```

#### Calico Cloud Provider

Calico目前只支持两个云提供商`AWS或GCE`，可以使用`calico_cloud_provider`设置。

**有效选项**

- `aws`
- `gce`

### Weave 网络插件选项

```yaml
network:
    plugin: weave
    weave_network_provider:
        password: "Q]SZOQ5wp@n$oijz"
```

#### Weave 加密

编织加密可以通过向网络提供程序配置传递字符串密码来启用。