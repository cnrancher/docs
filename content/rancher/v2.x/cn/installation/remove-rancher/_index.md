---
title: 初始化节点
weight: 9
---

将节点添加到群集时后，会创建容器、虚拟网络接口等资源和证书、配置文件。从群集中正常删除节点时(如果处于Active状态)，将自动清除这些资源，并且只需重新启动节点即可。当节点无法访问且无法使用自动清理，或者异常导致节点脱离集群后，如果需要再次将节点加入集群，那么需要手动进行节点初始化操作。

## 一、通过Rancher UI从群集中删除节点

当节点处于Active状态时，从群集中删除节点将触发清理节点的进程。完成自动清理过程后，需要重新启动节点，以确保正确删除所有非持久性数据。

- 如何重新启动节点？

    ```bash
    #using reboot
    sudo reboot
    #using shutdown
    sudo shutdown -r now
    ```

## 二、手动清理节点

当节点无法访问并已从群集中删除，将无法触发自动清理过程。需按照以下步骤手动清理节点:

> **警告:** 下面操作的命令将删除节点中的数据，在执行任何命令之前，请确保已进行数据备份。

### 1、Docker容器、镜像、容器卷

- 清理所有Docker容器和容器卷

  ```bash
  docker rm -f $(docker ps -qa)
  docker volume rm $(docker volume ls -q)
  ```

- 如果需要清理Docker镜像,执行以下命令

  ```bash
  docker rmi -f $(docker images -q)
  ```

### 2、Mounts挂载

在运行Pod后，会有很多挂载文件。比如:

|  挂载路径                    |
| --------------------------- |
|`/var/lib/kubelet/pods/XXX`  |
| `/var/lib/kubelet`          |
| `/var/lib/rancher`          |

- 卸载挂载文件

```bash
for mount in $(mount | grep tmpfs | grep '/var/lib/kubelet' | awk '{ print $3 }') /var/lib/kubelet /var/lib/rancher; do umount $mount; done
```

### 3、目录和文件

将节点添加到群集时，会自动创建以下目录，应将其删除。

| 目录                         |
| ---------------------------- |
| `/etc/ceph`                  |
| `/etc/cni`                   |
| `/etc/kubernetes`            |
| `/opt/cni`                   |
| `/opt/rke`                   |
| `/run/secrets/kubernetes.io` |
| `/run/calico`                |
| `/run/flannel`               |
| `/var/lib/calico`            |
| `/var/lib/etcd`              |
| `/var/lib/cni`               |
| `/var/lib/kubelet`           |
| `/var/lib/rancher`           |
| `/var/log/containers`        |
| `/var/log/pods`              |
| `/var/run/calico`            |

- 清理目录

```bash
rm -rf /etc/ceph \
       /etc/cni \
       /etc/kubernetes \
       /opt/cni \
       /opt/rke \
       /run/secrets/kubernetes.io \
       /run/calico \
       /run/flannel \
       /var/lib/calico \
       /var/lib/etcd \
       /var/lib/cni \
       /var/lib/kubelet \
       /var/lib/rancher \
       /var/log/containers \
       /var/log/pods \
       /var/run/calico
```

### 4、网络接口

- 网络名称

|接口名称                           |
|----------------------------------|
| `flannel.1`                      |
| `cni0`                           |
| `tunl0`                          |
| `caliXXXXXXXXXXX` (随机接口名称)  |
| `vethXXXXXXXX` (随机接口名称)     |

- 列出所有网络接口

```bash
# Using ip
ip address show

# Using ifconfig
ifconfig -a
```

- 手动删除网络接口

```bash
ip link delete interface_name
```

### 5、iptables规则

Iptables规则用于路由来自容器的流量。规则是动态创建的，不是持久的，重新启动节点会将iptables恢复到其原始状态。

| 链                                          |
| ------------------------------------------- |
| `cali-failsafe-in`                          |
| `cali-failsafe-out`                         |
| `cali-fip-dnat`                             |
| `cali-fip-snat`                             |
| `cali-from-hep-forward`                     |
| `cali-from-host-endpoint`                   |
| `cali-from-wl-dispatch`                     |
| `cali-fw-caliXXXXXXXXXXX` (随机链名称)    |
| `cali-nat-outgoing`                         |
| `cali-pri-kns.NAMESPACE` (每个命名空间链) |
| `cali-pro-kns.NAMESPACE` (每个命名空间链) |
| `cali-to-hep-forward`                       |
| `cali-to-host-endpoint`                     |
| `cali-to-wl-dispatch`                       |
| `cali-tw-caliXXXXXXXXXXX` (随机链名称)    |
| `cali-wl-to-host`                           |
| `KUBE-EXTERNAL-SERVICES`                    |
| `KUBE-FIREWALL`                             |
| `KUBE-MARK-DROP`                            |
| `KUBE-MARK-MASQ`                            |
| `KUBE-NODEPORTS`                            |
| `KUBE-SEP-XXXXXXXXXXXXXXXX` (随机链名称)  |
| `KUBE-SERVICES`                             |
| `KUBE-SVC-XXXXXXXXXXXXXXXX` (随机链名称)  |

- 如何列出所有iptables规则

```bash
iptables -L -t nat
iptables -L -t mangle
iptables -L
```
> 因为网络接口和iptables规则都不是永久存储的，所以建议在完成前面的步骤后重启主机。