---
title: 6 - 其他方法
weight: 6
---
>**警告** 本Chart目前属于测试阶段，你可以在测试或者开发环境使用。

Chart地址: https://github.com/xiaoluhong/server-chart.git

- 本Chart基于 https://github.com/rancher/server-chart/ 修改，当前支持版本为`rancher v2.1.7、v2.1.8、v2.2.0、v2.2.1`。
- 不支持LetsEncrypt、cert-manager提供证书，需手动通过Secret导入证书, 默认开启审计日志功能。

## 一、制作自签名证书或重命名权威认证证书

- 仓库根目录有一键创建自签名证书脚本，会自动创建`cacerts.pem`、`tls.key`、`tls.crt`。
- 如果使用权威认证证书，需要重命名crt和key为`tls.crt`和`tls.key`。

## 二、部署架构

### 1、内部ingress域名访问

通过集群内安装的ingress服务，使用七层域名转发来访问rancher server，请求流量将转发到rancher server容器的`80`端口。

- 把服务证书和CA证书作为密文导入K8S

> [Helm安装配置参考]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/helm-install/)

```bash
# 指定kubeconfig配置文件路径
kubeconfig=xxx
# 创建 cattle-system
kubectl --kubeconfig=$kubeconfig create namespace cattle-system
# 创建ssl证书密文
kubectl --kubeconfig=$kubeconfig -n cattle-system create \
secret tls tls-rancher-ingress --cert=./tls.crt --key=./tls.key
# 创建CA证书密文
kubectl --kubeconfig=$kubeconfig -n cattle-system create \
secret generic tls-ca --from-file=cacerts.pem
# 创建tiller serviceaccount
kubectl --kubeconfig=$kubeconfig -n kube-system create \
serviceaccount tiller
# 创建tiller clusterrolebinding
kubectl --kubeconfig=$kubeconfig create clusterrolebinding \
tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
# 安装tiller
helm_version=`helm version |grep Client | awk -F""\" '{print $2}'`
helm --kubeconfig=$kubeconfig init --skip-refresh --service-account \
tiller --tiller-image registry.cn-shanghai.aliyuncs.com/rancher/tiller:$helm_version
```

- 安装

```bash
git clone -b v2.1.7 https://github.com/xiaoluhong/server-chart.git

helm install  --kubeconfig=kube_config_xxx.yml \
  --name rancher \
  --namespace cattle-system \
  --set rancherImage=rancher/rancher \
  --set rancherRegistry=registry.cn-shanghai.aliyuncs.com \
  --set busyboxImage=rancher/busybox \
  --set hostname=<修改为自己的域名> \
  --set privateCA=true \
  server-chart/rancher
```

>**注意:** 1. 通过`--kubeconfig=`指定kubectl配置文件;\
>2. 如果使用权威ssl证书，则去除`--set privateCA=true`; \
>3. 如果为离线安装，可通过`rancherImage`指定镜像名称,`rancherRegistry`指定离线私有仓库地址;\
>4. 点击查看更多[Chart设置选项]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/rancher-install/chart-options/)。

### 2、主机NodePort访问(主机IP+端口)

有的场景需要使用IP去直接访问rancher server, 因为ingress默认不支持IP访问，所以这里禁用ingress。通过NodePort把rancher server容器端口映射到宿主机的端口上，这个时候rancher server容器作为ssl终止，请求流量转发到rancher server容器的`443`端口。

- 把服务证书和CA证书作为密文导入K8S

```bash
# 指定kubeconfig配置文件路径
kubeconfig=xxx
# 创建 cattle-system
kubectl --kubeconfig=$kubeconfig create namespace cattle-system
# 创建ssl证书密文
kubectl --kubeconfig=$kubeconfig -n cattle-system create \
secret tls tls-rancher-ingress --cert=./tls.crt --key=./tls.key
# 创建CA证书密文
kubectl --kubeconfig=$kubeconfig -n cattle-system create \
secret generic tls-ca --from-file=cacerts.pem
# 创建tiller serviceaccount
kubectl --kubeconfig=$kubeconfig -n kube-system create \
serviceaccount tiller
# 创建tiller clusterrolebinding
kubectl --kubeconfig=$kubeconfig create clusterrolebinding \
tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
# 安装tiller
helm_version=`helm version |grep Client | awk -F""\" '{print $2}'`
helm --kubeconfig=$kubeconfig init --skip-refresh --service-account \
tiller --tiller-image registry.cn-shanghai.aliyuncs.com/rancher/tiller:$helm_version
```

- 安装

```bash

git clone -b v2.1.7 https://github.com/xiaoluhong/server-chart.git

helm install  --kubeconfig=kube_config_xxx.yml \
  --name rancher \
  --namespace cattle-system \
  --set rancherImage=rancher/rancher \
  --set rancherRegistry=registry.cn-shanghai.aliyuncs.com \
  --set busyboxImage=rancher/busybox \
  --set service.type=NodePort \
  --set service.ports.nodePort=30303  \
  --set privateCA=true \
  server-chart/rancher
```

>**注意:** 1. 通过`--kubeconfig=`指定kubectl配置文件; \
>2. 如果使用权威ssl证书，则去除`--set privateCA=true`; \
>3. 如果为离线安装，可通过`rancherImage`指定镜像名称,`rancherRegistry`指定离线私有仓库地址; \
>4. 通过`--set service.ports.nodePort=30303`指定自己想要的端口; \
>5. 点击查看更多[Chart设置选项]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/rancher-install/chart-options/)。

### 3、外部七层负载均衡器+主机NodePort方式运行(禁用内部ingress)

有的场景，外部有七层负载均衡器作为ssl终止，常见用法是把负载均衡器的`443`端口代理到内部应用的`非https端口上，比如80`。为了保证网络转发性能，这里禁用了内置的ingress服务，以`NodePort方式`把rancher server容器的`80`端口映射到宿主机`30303`端口上。外部七层负载均衡器再把`443`端口反向代理到Rancher的NodePort端口上,请求流量将转发到rancher server容器的`80`端口。

- 把服务证书放在外部负载均衡器上，比如nginx.nginx配置参考: [NGIN示例配置]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/rke-ha-install/https-l7/nginx/#1-创建nginx配置)

- 把CA证书作为密文导入K8S

```bash
# 指定kubeconfig配置文件路径
kubeconfig=xxx
# 创建 cattle-system
kubectl --kubeconfig=$kubeconfig create namespace cattle-system
# 创建ssl证书密文
kubectl --kubeconfig=$kubeconfig -n cattle-system create \
secret tls tls-rancher-ingress --cert=./tls.crt --key=./tls.key
# 创建CA证书密文
kubectl --kubeconfig=$kubeconfig -n cattle-system create \
secret generic tls-ca --from-file=cacerts.pem
# 创建tiller serviceaccount
kubectl --kubeconfig=$kubeconfig -n kube-system create \
serviceaccount tiller
# 创建tiller clusterrolebinding
kubectl --kubeconfig=$kubeconfig create clusterrolebinding \
tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
# 安装tiller
helm_version=`helm version |grep Client | awk -F""\" '{print $2}'`
helm --kubeconfig=$kubeconfig init --skip-refresh --service-account \
tiller --tiller-image registry.cn-shanghai.aliyuncs.com/rancher/tiller:$helm_version
```

- 安装

```bash
git clone -b v2.1.7 https://github.com/xiaoluhong/server-chart.git

helm install  --kubeconfig=kube_config_xxx.yml \
  --name rancher \
  --namespace cattle-system \
  --set rancherImage=rancher/rancher \
  --set rancherRegistry=registry.cn-shanghai.aliyuncs.com \
  --set busyboxImage=rancher/busybox \
  --set service.type=NodePort \
  --set service.ports.nodePort=30303 \
  --set tls=external \
  --set privateCA=true \
  server-chart/rancher
```

>**注意:** 1. 通过`--kubeconfig=`指定kubectl配置文件; \
>2. 如果使用权威ssl证书，则去除`--set privateCA=true`; \
>3. 如果为离线安装，可通过`rancherImage`指定镜像名称,`rancherRegistry`指定离线私有仓库地址; \
>4. 通过`--set service.ports.nodePort=30303`指定自己想要的端口; \
>5. 点击查看更多[Chart设置选项]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/tcp-l4/rancher-install/chart-options/)。
