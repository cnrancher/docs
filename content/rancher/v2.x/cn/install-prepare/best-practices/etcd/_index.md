---
title: 2 - ETCD调优
weight: 2
---

## 一、磁盘IOPS

etcd对磁盘写入延迟非常敏感，通常需要50顺序写入IOPS(例如: 7200RPM磁盘)。对于负载较重的集群，建议使用500顺序写入IOPS(例如，典型的本地SSD或高性能虚拟化块设备)。请注意，大多数云服务器或者云存储提供并发IOPS而不是顺序IOPS，提供的并发IOPS可能比顺序IOPS大10倍。为了测量实际的顺序IOPS，建议使用磁盘基准测试工具，如[diskbench](https://github.com/ongardie/diskbenchmark)或[fio](https://github.com/axboe/fio)。

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

## 二、磁盘IO优先级

由于etcd必须将数据持久保存到磁盘日志文件中，因此来自其他进程的磁盘活动可能会导致增加`写入时间`，结果可能会导致etcd请求超时和临时`leader`丢失。当给定高磁盘优先级时，etcd服务可以稳定地与这些进程一起运行。

在Linux上，etcd的磁盘优先级可以配置为ionice：

```bash
sudo ionice -c2 -n0 -p $(pgrep etcd)
```

>**温馨提示**: 因为主机重启或者容器重启后，容器中进程的PID会发生变化，所以建议把以上命令放在系统的启动脚本中（比如Ubuntu的`/etc/init.d/rc.local`脚本中），并且把命令配置在crontab定时任务中。

## 三、修改空间配额大小

默认ETCD空间配额大小为2G，超过2G将不再写入数据。通过给ETCD配置`--quota-backend-bytes`参数增大空间配额,最大支持8G。

> RKE或者Rancher UI自定义部署集群的时候，在yaml文件中指定以下参数

```yaml
services:
  etcd:
    # 开启自动备份
    ## rke版本大于等于0.2.x或rancher版本大于等于2.2.0时使用
    backup_config:
      enabled: true
      interval_hours: 12
      retention: 6
    ## rke版本小于0.2.x或rancher版本小于2.2.0时使用
    snapshot: true
    creation: 5m0s
    retention: 24h
    # 修改空间配额为$((6*1024*1024*1024))，默认2G,最大8G
    extra_args:
      quota-backend-bytes: '6442450944'
      auto-compaction-retention: 240 #(单位小时)
```

- 磁盘碎片整理

通过`auto-compaction-retention`对历史数据压缩后，后端数据库可能会出现内部碎片。内部碎片是指空闲状态的，能被后端使用但是仍然消耗存储空间，碎片整理过程将此存储空间释放回文件系统。

要对etcd进行碎片整理，需手动在etcd容器中执行以下命令：

```bash
etcdctl defrag

Finished defragmenting etcd member[127.0.0.1:2379]
```

- [释放数据库空间]({{< baseurl >}}/rancher/v2.x/cn/faq/etcd/compact)

## 四、网络延迟

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

> 根据实际情况修改接口名称
