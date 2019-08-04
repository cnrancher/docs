---
title: 1 - 主机OS调优
weight: 1
---

## 一、内核调优

```bash
echo "
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.ipv4.conf.all.forwarding = 1
net.ipv4.neigh.default.gc_thresh1 = 4096
net.ipv4.neigh.default.gc_thresh2 = 6144
net.ipv4.neigh.default.gc_thresh3 = 8192
net.ipv4.neigh.default.gc_interval=60
net.ipv4.neigh.default.gc_stale_time=120
" >> /etc/sysctl.conf
```

接着执行`sysctl -p`

## 二、nofile

```bash
cat >> /etc/security/limits.conf <<EOF
* soft nofile 65535
* hard nofile 65536
EOF
```
