---
title: 备份和恢复
weight: 4
---

本节介绍如何创建Rancher数据备份以及如何在灾难情况下进行数据恢复。

- [数据备份](./backups/)
- [数据恢复](./restorations/)

## ETCD集群容错表

建议在ETCD集群中使用奇数个成员,通过添加额外成员可以获得更高的失败容错，具体详情可以查阅[optimal-cluster-size](https://coreos.com/etcd/docs/latest/v2/admin_guide.html#optimal-cluster-size)

| 集群大小 | MAJORITY | 失败容错 |
| ------------ | -------- | ----------------- |
| 1            | 1        | 0                 |
| 2            | 2        | 0                 |
| 3            | 2        | **1**             |
| 4            | 3        | 1                 |
| 5            | 3        | **2**             |
| 6            | 4        | 2                 |
| 7            | 4        | **3**             |
| 8            | 5        | 3                 |
| 9            | 5        | **4**             |