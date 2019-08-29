---
title: 7 - 清理节点
weight: 7
aliases:
  - /docs/remove-node
---

将节点添加到集群时后，会创建容器、虚拟网络接口等资源和证书、配置文件。从集群中正常删除节点时(如果处于Active状态)，将自动清除这些资源，并且只需重新启动节点即可。当节点无法访问且无法使用自动清理，或者异常导致节点脱离集群后，如果需要再次将节点加入集群，那么需要手动进行节点初始化操作。

## 手动清理节点

> **警告:** 以下操作将删除节点中的数据，在执行命令之前，请确保已进行数据备份。

```bash
  # 停止服务
  systemctl  disable kubelet.service
  systemctl  disable kube-scheduler.service
  systemctl  disable kube-proxy.service
  systemctl  disable kube-controller-manager.service
  systemctl  disable kube-apiserver.service

  systemctl  stop kubelet.service
  systemctl  stop kube-scheduler.service
  systemctl  stop kube-proxy.service
  systemctl  stop kube-controller-manager.service
  systemctl  stop kube-apiserver.service

  # 删除所有容器
  docker rm -f $(docker ps -qa)

  # 删除所有容器卷
  docker volume rm $(docker volume ls -q)

  # 卸载mount目录
  for mount in $(mount | grep tmpfs | grep '/var/lib/kubelet' | awk '{ print $3 }') /var/lib/kubelet /var/lib/rancher; do umount $mount; done
  
  # 备份目录
  mv /etc/kubernetes /etc/kubernetes-bak-$(date +"%Y%m%d%H%M")
  mv /var/lib/etcd /var/lib/etcd-bak-$(date +"%Y%m%d%H%M")
  mv /var/lib/rancher /var/lib/rancher-bak-$(date +"%Y%m%d%H%M")
  mv /opt/rke /opt/rke-bak-$(date +"%Y%m%d%H%M")

  # 删除残留路径
  rm -rf /etc/ceph \
       /etc/cni \
       /opt/cni \
       /run/secrets/kubernetes.io \
       /run/calico \
       /run/flannel \
       /var/lib/calico \
       /var/lib/cni \
       /var/lib/kubelet \
       /var/log/containers \
       /var/log/pods \
       /var/run/calico

  # 清理网络接口
  network_interface=`ls /sys/class/net`
  for net_inter in $network_interface;
  do
    if ! echo $net_inter | grep -qiE 'lo|docker0|eth*|ens*';then
      ip link delete $net_inter
    fi
  done

  # 清理残留进程
  port_list='80 443 6443 2376 2379 2380 8472 9099 10250 10254'

  for port in $port_list
  do
    pid=`netstat -atlnup|grep $port |awk '{print $7}'|awk -F '/' '{print $1}'|grep -v -|sort -rnk2|uniq`
    if [[ -n $pid ]];then
      kill -9 $pid
    fi
  done

  pro_pid=`ps -ef |grep -v grep |grep kube|awk '{print $2}'`
  
  if [[ -n $pro_pid ]];then
    kill -9 $pro_pid
  fi

  # 清理Iptables表
  ## 注意：如果节点Iptables有特殊配置，以下命令请谨慎操作
  sudo iptables --flush
  sudo iptables --flush --table nat
  sudo iptables --flush --table filter
  sudo iptables --table nat --delete-chain
  sudo iptables --table filter --delete-chain

  systemctl restart docker
```
