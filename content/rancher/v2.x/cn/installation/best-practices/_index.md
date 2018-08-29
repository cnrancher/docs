---
title: 11 - 最佳实践
weight: 11
---

## 一、etcd

### 1、磁盘IOPS

etcd对磁盘写入延迟非常敏感，通常需要50顺序IOPS(例如: 7200RPM磁盘)。对于负载较重的群集，建议使用500连续IOPS(例如，典型的本地SSD或高性能虚拟化块设备)。请注意，大多数云提供程序发布并发IOPS而不是顺序IOPS; 发布的并发IOPS可能比顺序IOPS大10倍。为了测量实际的顺序IOPS，建议使用磁盘基准测试工具，如[diskbench](https://github.com/ongardie/diskbenchmark)或[fio](https://github.com/axboe/fio)。

>**PS** 常见磁盘平均物理寻道时间约为: \
7200转/分的STAT硬盘平均物理寻道时间是9ms \
10000转/分的STAT硬盘平均物理寻道时间是6ms \
15000转/分的SAS硬盘平均物理寻道时间是4ms
>
>常见硬盘的旋转延迟时间约为: \
7200  rpm的磁盘平均旋转延迟大约为60X1000/7200/2=4.17ms \
10000 rpm的磁盘平均旋转延迟大约为60X1000/10000/2=3ms，\
15000 rpm的磁盘其平均旋转延迟约为60X1000/15000/2=2ms。
>
>最大IOPS的理论计算方法:\
IOPS=1000ms/(寻道时间+旋转延迟)。忽略数据传输时间。\
7200 rpm的磁盘IOPS=1000/(9+4.17)=76IOPS\
10000 rpm的磁盘IOPS=1000/(6+3)=111IOPS\
15000 rpm的磁盘IOPS=1000/(4+2)=166IOPS

### 2、磁盘IO优先级

由于etcd必须将提议持久保存到日志中，因此来自其他进程的磁盘活动可能会导致延迟`fsync`时间。结果是etcd可能会错过心跳，导致请求超时和临时`leader`丢失。当给定高磁盘优先级时，etcd服务可以稳定地与这些进程一起运行。

在Linux上，etcd的磁盘优先级可以配置为ionice：

```bash
sudo ionice -c2 -n0 -p `pgrep etcd`
```

### 3、修改空间配额大小

默认ETCD空间配额大小为2G，超过2G将不再写入数据。通过给ETCD配置`--quota-backend-bytes`参数增大空间配额,最大支持8G。

### 4、网络延迟

如果有大量并发客户端请求etcd leader服务，则可能由于网络拥塞而延迟处理`follower`对等请求。在`follower`节点上的发送缓冲区错误消息：

```bash
dropped MsgProp to 247ae21ff9436b2d since streamMsg's sending buffer is full
dropped MsgAppResp to 247ae21ff9436b2d since streamMsg's sending buffer is full
```

可以通过在客户端提高etcd对等网络流量优先级来解决这些错误。在Linux上，可以使用流量控制机制对对等流量进行优先级排序：

```bash
tc qdisc add dev eth0 root handle 1: prio bands 3
tc filter add dev eth0 parent 1: protocol ip prio 1 u32 match ip sport 2380 0xffff flowid 1:1
tc filter add dev eth0 parent 1: protocol ip prio 1 u32 match ip dport 2380 0xffff flowid 1:1
tc filter add dev eth0 parent 1: protocol ip prio 2 u32 match ip sport 2739 0xffff flowid 1:1
tc filter add dev eth0 parent 1: protocol ip prio 2 u32 match ip dport 2739 0xffff flowid 1:1
```

>根据实际情况修改接口名称

## 二、主机/OS

### 1、增加ARP缓存大小

```bash
cat >>  /etc/sysctl.conf <<EOF
net.ipv4.neigh.default.gc_thresh1=<value1>
net.ipv4.neigh.default.gc_thresh2=<value2>
net.ipv4.neigh.default.gc_thresh3=<value3>
EOF
```

接着执行`sysctl -p`

> 根据主机资源大小来调整<value>值.

## 三、Docker

- max-concurrent-downloads,max-concurrent-uploads

    Docker镜像下载最大并发数

```bash
touch /etc/docker/daemon.json
cat > /etc/docker/daemon.json <<EOF
{
"max-concurrent-downloads": 10,
"max-concurrent-uploads": 10,
"registry-mirrors": ["https://7bezldxe.mirror.aliyuncs.com"],
"storage-driver": "overlay2",
"storage-opts": [
"overlay2.override_kernel_check=true"
]
}
EOF
systemctl daemon-reload && systemctl restart docker
```