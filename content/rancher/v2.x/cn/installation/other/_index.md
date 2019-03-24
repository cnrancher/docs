---
title: 6 - 其他方法
weight: 6
---
>**警告** 本Chart目前属于测试阶段，你可以在测试或者开发环境使用。

Chart地址: https://github.com/xiaoluhong/server-chart.git

本Chart基于 https://github.com/rancher/server-chart/tree/2019.3.1 修改，当前版本为`rancherv2.1.7`。不支持LetsEncrypt、cert-manager提供证书，需手动通过Secret导入证书, 默认开启审计日志功能。

## 一、制作自签名证书或重命名权威认证证书

- 仓库根目录有一键创建自签名证书脚本，会自动创建`cacerts.pem`、`tls.key`、`tls.crt`。
- 如果使用权威认证证书，需要重命名crt和key为`tls.crt`和`tls.key`。

## 二、部署架构

### 1、内部ingress域名访问

通过集群内安装的ingress服务，使用七层域名转发来访问rancher server，请求流量将转发到rancher server容器的`80`端口。

- 把服务证书和CA证书作为密文导入K8S

```bash
# 指定kubeconfig配置文件路径
kubeconfig=xxx

kubectl --kubeconfig=$kubeconfig create namespace cattle-system
kubectl --kubeconfig=$kubeconfig -n cattle-system create secret tls tls-rancher-ingress --cert=./tls.crt --key=./tls.key
kubectl --kubeconfig=$kubeconfig -n cattle-system create secret generic tls-ca --from-file=cacerts.pem
```

- 安装

```bash
git clone -b v2.1.7 https://github.com/xiaoluhong/server-chart.git

helm install  --kubeconfig=kube_config_xxx.yml \
  --name rancher \
  --namespace cattle-system \
  --set hostname=<修改为您自己的域名> \
  --set service.type=ClusterIP \
  --set ingress.tls.source=secret \
  --set rancherImage=rancher/rancher \
  --set rancherImageTag=v2.1.7 \
  server-chart/rancher
```

>**注意:** 1. 通过`--kubeconfig=`指定kubectl配置文件;\
>2. 如果要把外部负载均衡器作为ssl终止，需添加参数: `--set tls=external`;\
>3. 如果使用自签名证书，需要设置参数: `--set privateCA=true`;\
>4. 如果为离线安装，可通过`rancherImage`指定镜像名称;\
>5. 默认安装`v2.1.7`版本，可通过`rancherImageTag`更换镜像版本;\
>6. 点击查看更多[Chart设置选项]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/rancher-install/chart-options/)。

### 2、主机NodePort访问(主机IP+端口)

有的场景需要使用IP去直接访问rancher server, 因为ingress默认不支持IP访问，所以这里禁用ingress。通过NodePort把rancher server容器端口映射到宿主机的端口上，这个时候rancher server容器作为ssl终止，请求流量转发到rancher server容器的`443`端口。

- 把服务证书和CA证书作为密文导入K8S

```bash
# 指定kubeconfig配置文件路径 
kubeconfig=xxx

kubectl --kubeconfig=$kubeconfig create namespace cattle-system
kubectl --kubeconfig=$kubeconfig -n cattle-system create secret tls tls-rancher-ingress --cert=./tls.crt --key=./tls.key
kubectl --kubeconfig=$kubeconfig -n cattle-system create secret generic tls-ca --from-file=cacerts.pem
```

- 安装

```bash
git clone -b v2.1.7 https://github.com/xiaoluhong/server-chart.git

helm install  --kubeconfig=kube_config_xxx.yml \
  --name rancher \
  --namespace cattle-system \
  --set rancherImage=rancher/rancher \
  --set rancherImageTag=v2.1.7 \
  --set service.type=NodePort \
  --set ingress.tls.source=secret \
  --set service.ports.nodePort=30303  \
  server-chart/rancher
```

>**注意:** 1. 通过`--kubeconfig=`指定kubectl配置文件;\
>2. 如果要把外部负载均衡器作为ssl终止，需添加参数: `--set tls=external`;\
>3. 如果使用自签名证书，需要设置参数: `--set privateCA=true`;\
>4. 如果为离线安装，可通过`rancherImage`指定镜像名称;\
>5. 默认安装`v2.1.7`版本，可通过`rancherImageTag`更换镜像版本;\
>6. 点击查看更多[Chart设置选项]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/rancher-install/chart-options/)。

### 3、外部七层负载均衡器+主机NodePort方式运行(禁用内部ingress)

有的场景，外部有七层负载均衡器作为ssl终止，常见用法是把负载均衡器的`443`端口代理到内部应用的`非https端口上，比如80`。为了保证网络转发性能，这里禁用了内置的ingress服务，以`NodePort方式`把rancher server容器的`80`端口映射到宿主机`30303`端口上。外部七层负载均衡器再把`443`端口反向代理到Rancher的NodePort端口上,请求流量将转发到rancher server容器的`80`端口。

- 把服务证书放在外部负载均衡器上，比如nginx.nginx配置参考: [NGIN示例配置]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/rke-ha-install/https-l7/nginx/#1-创建nginx配置)

- 把CA证书作为密文导入K8S

```bash
# 指定kubeconfig配置文件路径 
kubeconfig=xxx

kubectl --kubeconfig=$kubeconfig create namespace cattle-system
kubectl --kubeconfig=$kubeconfig -n cattle-system create secret generic tls-ca --from-file=cacerts.pem
```

- 安装

```bash
git clone -b v2.1.7 https://github.com/xiaoluhong/server-chart.git

helm install  --kubeconfig=kube_config_xxx.yml \
  --name rancher \
  --namespace cattle-system \
  --set service.type=NodePort \
  --set tls=external  \
  --set rancherImage=rancher/rancher \
  --set rancherImageTag=v2.1.7 \
  --set service.ports.nodePort=30303  \
  server-chart/rancher
```

>**注意:** 1. 通过`--kubeconfig=`指定kubectl配置文件;\
>2. 如果使用自签名证书，需要设置参数: `--set privateCA=true`;\
>3. 如果为离线安装，可通过`rancherImage`指定镜像名称;\
>4. 默认安装`v2.1.7`版本，可通过`rancherImageTag`更换镜像版本;\
>5. 点击查看更多[Chart设置选项]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/rancher-install/chart-options/)。

## 三、Chart版本

```bash
NAME                      CHART VERSION    APP VERSION    DESCRIPTION
rancher-stable/rancher    2018.3.1           v2.1.7      Install Rancher Server to manage Kubernetes clusters acro...
```
