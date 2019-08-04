---
title: 4 - kubernetes调优
weight: 4
---

## kube-apiserver

> RKE或者Rancher UI自定义部署集群的时候，在yaml文件中指定以下参数

```yaml
services:
  kube-api:
    extra_args:
```

## kube-controller

> RKE或者Rancher UI自定义部署集群的时候，在yaml文件中指定以下参数

```yaml
services:
  kube-controller:
    extra_args:
      # 修改每个节点子网大小(cidr掩码长度)，默认为24，可用IP为254个；23，可用IP为510个；22，可用IP为1022个；
      node-cidr-mask-size: 24
      # 控制器定时与节点通信以检查通信是否正常，周期默认5s
      node-monitor-period: '5s'
      # 当节点通信失败后，再等一段时间kubernetes判定节点为notready状态。
      ## 这个时间段必须是kubelet的nodeStatusUpdateFrequency(默认10s)的N倍，
      ## 其中N表示允许kubelet同步节点状态的重试次数，默认40s。
      node-monitor-grace-period: '20s'
      # 再持续通信失败一段时间后，kubernetes判定节点为unhealthy状态，默认1m0s。
      node-startup-grace-period: '30s'
      # 再持续失联一段时间，kubernetes开始迁移失联节点的Pod，默认5m0s。
      pod-eviction-timeout: '1m'
```

## kubelet

> RKE或者Rancher UI自定义部署集群的时候，在yaml文件中指定以下参数

```yaml
services:
  kubelet:
    extra_args:
      # 修改节点最大Pod数量,默认110个
      max-pods: '250'
      # 密文和配置映射同步时间，默认1分钟
      sync-frequency: '3s'
      # 自定义pause镜像,默认为rancher/pause:3.1
      pod-infra-container-image: 'rancher/pause:3.1'
      # 传递给网络插件的MTU值，以覆盖默认值，设置为0(零)则使用默认的1460
      network-plugin-mtu: '1500'
      # Kubelet进程可以打开的文件数（默认1000000）,根据节点配置情况调整
      max-open-files: '2000000'
      # 与apiserver会话时的并发数，默认是10
      kube-api-burst: '30'
      # 与apiserver会话时的 QPS,默认是5
      kube-api-qps: '15'
      # kubelet默认一次拉取一个镜像，设置为false可以同时拉取多个镜像，
      ## 前提是存储驱动要为overlay2，对应的Dokcer也需要增加下载并发数，参考：三、Docker
      serialize-image-pulls: 'false'
      # 拉取镜像的最大并发数，registry-burst不能超过registry-qps ，
      ## 仅当registry-qps大于0(零)时生效，(默认10)。如果registry-qps为0则不限制(默认5)。
      registry-burst: '10'
      registry-qps: '0'

      cgroups-per-qos: 'true'
      cgroup-driver: 'cgroupfs'

      # enforce-node-allocatable: 'pods,kube-reserved,system-reserved'
      enforce-node-allocatable: 'pods'

      # system-reserved: 'cpu=1,memory=1Gi,ephemeral-storage=10Gi'
      system-reserved: 'cpu=0.25,memory=200Mi'

      # kube-reserved: 'cpu=1,memory=1Gi,ephemeral-storage=10Gi'
      kube-reserved: 'cpu=0.25,memory=1500Mi'

      # 设置进行pod驱逐的阈值，这个参数只支持内存和磁盘。通过--eviction-hard标志预留一些内存后，当节点上的可用内存降至保留值以下时，kubelet 将会对pod进行驱逐。
      # eviction-hard: 'memory.available<300Mi,nodefs.available<10%,imagefs.available<15%,nodefs.inodesFree<5%'
      eviction-hard: 'memory.available<100Mi'

      eviction-soft: 'memory.available<0.5Gi'
      eviction-soft-grace-period: 'memory.available=1m30s'
      eviction-max-pod-grace-period: '30'
      eviction-pressure-transition-period: '30s'

```

## kube-proxy

## kube-scheduler
