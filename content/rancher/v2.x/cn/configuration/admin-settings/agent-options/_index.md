---
title: 11 - Rancher Agent设置选项
weight: 11
---

Rancher在每个节点上部署代理以与节点通信。 此页面描述了可以传递给代理的选项，要使用这些选项，您需要采用[创建自定义集群]({{< baseurl >}}/rancher/v2.x/cn/cluster-provisioning/custom-clusters/) ，并在`docker run`添加节点时将选项添加到生成的命令中。

## 常规选项

| 参数  | 环境变量 | 描述 |
| ---------- | -------------------- | ----------- |
| `--server` | `CATTLE_SERVER` | Rancher配置的url地址 |
| `--token`  | `CATTLE_TOKEN` | 在Rancher中注册节点所需的令牌，可以在`https://<rancher_url>/v3/clusters/<Cluster-ID>/clusterregistrationtokens`页面查询 |
| `--ca-checksum`  | `CATTLE_CA_CHECKSUM` | Rancher配置的`cacerts`的SHA256校验和，可以在`https://<rancher_url>/v3/clusters/<Cluster-ID>/clusterregistrationtokens`页面查询 |
| `--node-name` | `CATTLE_NODE_NAME` | 用于覆盖注册节点原有的主机名(默认取: hostname -s) |
| `--label` | `CATTLE_NODE_LABEL` | 向节点添加标签 (`--label key=value`) |

## 角色选项

| 参数  | 环境变量 | 描述 |
| ---------- | -------------------- | ----------- |
| `--all-roles` | `ALL=true` | 给节点指定所有角色 (`etcd`,`controlplane`,`worker`) |
| `--etcd`  | `ETCD=true` | 指定`etcd`角色 |
| `--controlplane`  | `CONTROL=true` | 指定 `controlplane` 角色 |
| `--worker`  | `WORKER=true` | 指定 `worker` 角色 |

## IP地址选项

| 参数  | 环境变量 | 描述 |
| ---------- | -------------------- | ----------- |
| `--address` | `CATTLE_ADDRESS` | 节点注册使用的IP地址 |
| `--internal-address` | `CATTLE_INTERNAL_ADDRESS` | 专用网络上用于主机间通信的IP地址 |

### 动态IP地址选项

出于自动化目的，您不能在命令中具有特定的IP地址。为此，我们有动态IP地址选项。它们支持 `--address` 和 `--internal-address`。

| 值  | 示例 | 描述 |
| ---------- | -------------------- | ----------- |
| 接口名称 | `--address eth0` | 将从给定接口检索配置的第一个IP地址                           |
| `ipify` | `--address ipify` | 将使用从`https://api.ipify.org`中检索的值 |
| `awslocal` | `--address awslocal` | 将使用从`http://169.254.169.254/latest/meta-data/local-ipv4`中检索的值 |
| `awspublic` | `--address awspublic` | 将使用从 `http://169.254.169.254/latest/meta-data/public-ipv4` 检索的值 |
| `doprivate` | `--address doprivate` | 将使用从 `http://169.254.169.254/metadata/v1/interfaces/private/0/ipv4/address` 检索的值 |
| `dopublic` | `--address dopublic` | 将使用从 `http://169.254.169.254/metadata/v1/interfaces/public/0/ipv4/address` 检索的值 |
| `azprivate` | `--address azprivate` | 将使用从`http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/privateIpAddress?api-version=2017-08-01&format=text` 检索的值 |
| `azpublic` | `--address azpublic` | 将使用从`http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/publicIpAddress?api-version=2017-08-01&format=text` 检索的值 |
| `gceinternal` | `--address gceinternal` | 将使用从 `http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip` 检索的值 |
| `gceexternal` | `--address gceexternal` | 将使用从`http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/access-configs/0/external-ip` 检索的值 |
| `packetlocal` | `--address packetlocal` | 将使用从 `https://metadata.packet.net/2009-04-04/meta-data/local-ipv4` 检索的值 |
| `packetpublic` | `--address packetlocal` | 将使用从 `https://metadata.packet.net/2009-04-04/meta-data/public-ipv4` 检索的值 |
