---
title: 4 - 主机驱动RKE集群
weight: 4
---

像Digitalocean这种未提供云Kubernetes服务的云平台，或者像vsphere这种本地基础架构平台。除了手动创建虚拟机然后通过执行自定义命令方式来创建集群外，还可以通过主机驱动对接平台API自动创建虚拟机，然后通过`RKE工具`自动安装Kubernetes集群，我们把这种集群叫做`主机驱动RKE集群`。
