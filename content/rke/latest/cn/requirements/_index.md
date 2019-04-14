---
title: 1 - 需求
weight: 1
---

## 一、操作系统

几乎所有安装了Docker的Linux操作系统都可以运行RKE，但是推荐使用Ubuntu 16.04，因为你大多数RKE的开发和测试都在Ubuntu 16.04上。

某些操作系统有限制和特定要求:

- [SSH user]({{< baseurl >}}/rke/latest/cn/config-options/nodes/#ssh-users) - 用于访问节点的SSH用户,必须加入docker组:

   ```bash
   usermod -aG docker <user_name>
   ```

   请参阅[Manage Docker as a non-root user](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user) 以了解如何在不使用root用户的情况下配置对Docker的访问。

- 应在工作节点上禁用交换

- 应该加载以下内核模块。可以使用以下方法检查：
   * `modprobe module_name`
   * `lsmod | grep module_name`
   * `grep module_name /lib/modules/$(uname -r)/modules.builtin`, 如果它是一个内置模块

Module name |
------------|
br_netfilter |
ip6_udp_tunnel |
ip_set |
ip_set_hash_ip |
ip_set_hash_net |
iptable_filter |
iptable_nat |
iptable_mangle |
iptable_raw |
nf_conntrack_netlink |
nf_conntrack |
nf_conntrack_ipv4 |
nf_defrag_ipv4 |
nf_nat |
nf_nat_ipv4 |
nf_nat_masquerade_ipv4 |
nfnetlink |
udp_tunnel |
veth |
vxlan |
x_tables |
xt_addrtype |
xt_conntrack |
xt_comment |
xt_mark |
xt_multiport |
xt_nat |
xt_recent |
xt_set |
xt_statistic |
xt_tcpudp |

- 必须应用以下sysctl设置

```bash
net.bridge.bridge-nf-call-iptables=1
```

### Red Hat Enterprise Linux(RHEL)/Oracle Enterprise Linux(OEL)/CentOS

如果使用Red Hat Enterprise Linux，Oracle Enterprise Linux或CentOS，由于[Bugzilla 1527565](https://bugzilla.redhat.com/show_bug.cgi?id=1527565)您无法将root用户用作SSH用户。请根据您在节点上安装Docker的方式，按照以下说明正确设置Docker。

#### 使用docker-ce

检查是否安装docker-ce或docker-ee，可以执行以下命令检查已安装的软件包:

```bash
rpm -q docker-ce
```

#### 使用RHEL/CentOS维护的Docker

如果您使用的是Red Hat/CentOS提供的Docker软件包，则软件包名称为docker。您可以执行以下命令检查已安装的软件包

```bash
rpm -q docker
```

如果您使用的是Red Hat/CentOS提供的Docker软件包，该`dockerroot`组将自动添加到系统中。您需要编辑（或创建）`/etc/docker/daemon.json`以包含以下内容:

```json
{
    "group": "dockerroot"
}
```

编辑或创建文件后重新启动Docker,重新启动Docker后，您可以检查Docker socket（/var/run/docker.sock）的组权限，该权限应显示为group(dockerroot)

```bash
srw-rw----. 1 root dockerroot 0 Jul  4 09:57 /var/run/docker.sock
```

将要使用的SSH用户添加到该组，这不是root用户。

```bash
usermod -aG dockerroot <user_name>
```

要验证用户配置是否正确，请注销节点并使用SSH用户重新登录，然后执行`docker ps`：

```bash
ssh <user_name>@node
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

### Red Hat Atomic

在尝试将RKE与Red Hat Atomic节点一起使用之前，需要对操作系统进行一些更新才能使RKE正常工作。

#### OpenSSH 版本

默认情况下，Atomic安装OpenSSH 6.4，它不支持SSH隧道，这是核心RKE要求，需要升级openssh。

#### 创建Docker Group

默认情况下，Atomic不附带Docker组，可以通过启用特定用户来启动RKE来更新Docker套接字的所有权。

```bash
chown <user> /var/run/docker.sock
```

## 软件

- Docker - 每个Kubernetes版本都支持不同的Docker版本。

Kubernetes版本 | 支持Docker版本(s) |
----|----|
v1.13.x | RHEL Docker 1.13, 17.03.2, 18.06.2, 18.09.2 |
v1.12.x | RHEL Docker 1.13, 17.03.2, 18.06.2, 18.09.2 |
v1.11.x | RHEL Docker 1.13, 17.03.2, 18.06.2, 18.09.2 |

您可以按照[Docker安装](https://docs.docker.com/install/)说明操作，也可以使用Rancher的[安装脚本](https://github.com/rancher/install-docker)安装Docker。对于RHEL，请参阅[如何在Red Hat Enterprise Linux 7上安装Docker](https://access.redhat.com/solutions/3727511)。

Docker版本   | 安装脚本 |
----------|------------------
18.09.2 |  <code>curl https://releases.rancher.com/install-docker/18.09.2.sh &#124; sh</code> |
18.06.2 |  <code>curl https://releases.rancher.com/install-docker/18.06.2.sh &#124; sh</code> |
17.03.2 |  <code>curl https://releases.rancher.com/install-docker/17.03.2.sh &#124; sh</code> |

确认安装的docker版本: `docker version --format '{{.Server.Version}}'`

```bash
docker version --format '{{.Server.Version}}'
17.03.2-ce
```

- OpenSSH 7.0+ - 必须在每个节点上安装OpenSSH。

## 端口

{{< ports-rke-nodes >}}
{{< requirements_ports_rke >}}

If you are using an external firewall, make sure you have this port opened between the machine you are using to run `rke` and the nodes that you are going to use in the cluster.

### `iptables` 放行端口TCP/6443

```bash
# Open TCP/6443 for all
iptables -A INPUT -p tcp --dport 6443 -j ACCEPT

# Open TCP/6443 for one specific IP
iptables -A INPUT -p tcp -s your_ip_here --dport 6443 -j ACCEPT
```

### `firewalld`放行端口TCP/6443

```bash
# Open TCP/6443 for all
firewall-cmd --zone=public --add-port=6443/tcp --permanent
firewall-cmd --reload

# Open TCP/6443 for one specific IP
firewall-cmd --permanent --zone=public --add-rich-rule='
  rule family="ipv4"
  source address="your_ip_here/32"
  port protocol="tcp" port="6443" accept'
firewall-cmd --reload
```
