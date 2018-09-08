---
title: Kubernetes常见问题
---


## 1、部署Kubernetes时候出现以下有关cgroup的问题

```
Failed to get system container stats for "/system.slice/kubelet.service": 
failed to get cgroup stats for "/system.slice/kubelet.service": failed to 
get container info for "/system.slice/kubelet.service": unknown container 
"/system.slice/kubelet.service"
```

```
Expected state running but got error: Error response from daemon: 
oci runtime error: container_linux.go:247: starting container 
process caused "process_linux.go:258: applying cgroup configuration 
for process caused \"mountpoint for devices not found\""
```
以上问题为Kubernetes版本与docker 版本不兼容导致cgroup功能失效

## 2、Kubernetes  err: [nodes \"iZ2ze3tphuqvc7o5nj38t8Z\" not found]"

Rancher-Kubernetes中，节点之间通信需要通过hostname，如果没有内部DNS服务器，那么需要为每台节点配置hosts文件。

配置示例:假如某个节点主机名为node1,ip 地址为192.168.1.100

```
cat /etc/hosts<<EOF
127.0.0.1 localhost
192.168.1.100 node1
EOF
```

## 3、如何验证你的主机注册地址设置是否正确？

如果你正面临Rancher Agent和Rancher Server的连接问题，请检查主机设置。当你第一次尝试在UI中添加主机时，你需要设置主机注册的URL，该URL用于建立从主机到Rancher Server的连接。这个URL必须可以从你的主机访问到。为了验证它，你需要登录到主机并执行curl命令:

```
curl -i <Host Registration URL you set in UI>/v1
```
你应该得到一个json响应。 如果开启了认证，响应代码应为401。如果认证未打开，则响应代码应为200。

**注意:** 普通的HTTP请求和websocket连接(ws://)都将被使用。 如果此URL指向代理或负载平衡器，请确保它们可以支持Websocket连接。

## Kuberbetes UI 显示Service unavailable
很多同学正常部署Kuberbetes环境后无法进入Dashboard，基础设施应用栈均无报错。但通过查看 基础架构|容器 发现并没有Dashboard相关的容器.因为Kuberbetes在拉起相关服务(如Dashboard、内置DNS等服务)是通过应用商店里面的YML文件来定义的，YML文件中定义了相关的镜像名和版本。

而Rancher部署的Kuberbetes应用栈属于Kuberbetes的基础框架，相关的镜像通过dockerhub/rancher 仓库拉取。默认Rancher-catalog Kuberbetes YML中服务镜像都是从谷歌仓库拉取，在没有科学上网的情况下，国内环境几乎无法成功拉取镜像。

为了解决这一问题，优化中国区用户的使用体验，在RANCHER v1.6.11之前的版本，我们修改了 ```http://git.oschina.net/rancher/rancher-catalog```  仓库中的YML文件，将相关的镜像也同步到国内仓库，通过替换默认商店地址来实现加速部署；在RANCHER v1.6.11及之后的版本，不用替换商店catalog地址，直接通过在模板中定义仓库地址和命名空间就行实现加速；在后期的版本种，Kuberbetes需要的镜像都会同步到docker hub中。

安装方法见:[K8S加速安装](/blog/K8S-install)
