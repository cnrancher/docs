---
title: Rancher Agent 常见问题
---

## 1、Rancher Agent无法启动的原因是什么？

### 1.1、添加 `--name rancher-agent` (老版本)

如果您从UI中编辑`docker run .... rancher/agent...`命令并添加`--name rancher-agent`选项，那么Rancher Agent将启动失败。Rancher Agent在初始运行时会启动3个不同容器，一个是运行状态的，另外两个是停止状态的。Rancher Agent要成功连接到Rancher Server必须要有两个名字分别为`rancher-agent`和`rancher-agent-state`的容器，第三个容器是docker自动分配的名称，这个容器会被移除。

### 1.2、使用一个克隆的虚拟机

如果您使用了克隆其他Agent主机的虚拟机并尝试注册它，它将不能工作。在rancher-agent容器的日志中会产生`ERROR: Please re-register this agent.`字样的日志。Rancher主机的唯一ID保存在`/var/lib/rancher/state`，因为新添加和虚拟机和被克隆的主机有相同的唯一ID，所以导致无法注册成功。

解决方法是在克隆的VM上运行以下命令: 

```bash
rm -rf /var/lib/rancher/state; docker rm -fv rancher-agent; docker rm -fv rancher-agent-state
```

完成后可重新注册。

## 2、我在哪里可以找到Rancher agent容器的详细日志?

从v1.6.0起，在rancher-agent容器上运行`docker logs`将提供agent相关的所有日志。

## 3、主机是如何自动探测IP的？我该怎么去修改主机IP？如果主机IP改变了(因为重启)，我该怎么办？

当Agent连接Rancher Server时，它会自动检测Agent的IP。有时，自动探测的IP不是您想要使用的IP，或者选择了docker网桥的IP，如. `172.17.x.x`。
或者，您有一个已经注册的主机，当主机重启后获得了一个新的IP, 这个IP将会和Rancher UI中的主机IP不匹配。
您可以重新配置“CATTLE_AGENT_IP”设置，并将主机IP设置为您想要的。
当主机IP地址不正确时，容器将无法访问管理网络。要使主机和所有容器进入管理网络，只需编辑添加自定义主机的命令行，将新的IP指定为环境变量“CATTLE_AGENT_IP”。 在主机上运行编辑后的命令。 不要停止或删除主机上的现有的Rancher Agent容器！

```bash
sudo docker run -d -e CATTLE_AGENT_IP=<NEW_HOST_IP> --privileged \
-v /var/run/docker.sock:/var/run/docker.sock \
rancher/agent:v0.8.2 http://SERVER_IP:8080/v1/scripts/xxxx
```

## 4、错误提示如下:INFO: Attempting to connect to: http://192.168.xx.xx:8080/v1    ERROR: http://192.168.xx.xx:8080/v1 is not accessible (Failed to connect to 192.168.xx.xx port 8080: No route to host)

这个问题主要有以下几种情况:

1.Rancher Server服务器防火墙没有开通8080端口;

2.云平台安全组没有放行8080端口;

3.Agent 服务器没有开启IP转发规则 [为什么我的容器无法连接到网络?]:{site.baseurl}}/rancher/faqs/
troubleshooting/1为什么我的容器无法连接到网络;

=1为开启，=0为关闭

```bash
/etc/sysctl.conf
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

4.主机hosts(`/etc/hosts`)文件没有配置;

## 5、rancher下创建的服务容器，docker inspect 查看到Entrypoint和CMD后面有/.r/r字符，这个起什么作用？

./r 是基于weave wait编译出来的。
CNI网络下会添加/.r/r  这个参数，目的是:当容器启动时，其实网络设备还没设置好，这时候需要container 等待，不能启动真实业务，否则会失败。

## 6、Host not registered yet. Sleeping 1 second and trying again." Attempt=0 reportedUuid=752031dd-8c7e-4666-5f93-020d7f4da5d3

检查主机名和hosts配置, hosts中需要配置:

127.0.0.1 localhost

hostip  hostname 

## 7、Rancher cattle ， 为什么添加第二台主机后会把第一台主机替换掉？

在/var/lib/rancher/ 目录下保存了每个节点的注册配置信息。如果是克隆的主机，在/var/lib/rancher/ 目录有相同的内容，当新加主机的时候会识别为重新添加某台主机，这样之前添加的主机就会被替换掉。

解决办法:新加主机时，删除/var/lib/rancher/ 目录。
