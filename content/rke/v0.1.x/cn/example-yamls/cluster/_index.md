---
title: Cluster.ymls
weight: 300
---

有许多不同的[配置选项]({{< baseurl >}}/rke/v0.1.x/cn/config-options/)可以在集群配置文件中为RKE设置。以下是一些示例配置:

## 最小化`cluster.yml`示例

```yaml
nodes:
    - address: 1.2.3.4
      user: ubuntu
      role:
        - controlplane
        - etcd
        - worker
```

## 完整 `cluster.yml` 示例

```yaml
nodes:
    - address: 1.1.1.1
      user: ubuntu
      role:
        - controlplane
        - etcd
      ssh_key_path: /home/user/.ssh/id_rsa
      port: 2222
    - address: 2.2.2.2
      user: ubuntu
      role:
        - worker
      ssh_key: |-
        -----BEGIN RSA PRIVATE KEY-----

        -----END RSA PRIVATE KEY-----
    - address: example.com
      user: ubuntu
      role:
        - worker
      hostname_override: node3
      internal_address: 192.168.1.6
      labels:
        app: ingress

# 如果设置为true，将使用不受支持的Docker版本
ignore_docker_version: false

# 集群通信SSH私钥(private key)
# RKE将会以此私钥去连接集群节点
ssh_key_path: ~/.ssh/test

# Enable use of SSH agent to use SSH private keys with passphrase
# This requires the environment `SSH_AUTH_SOCK` configured pointing to your SSH agent which has the private key added
ssh_agent_auth: true

# 私有仓库
private_registries:
    - url: registry.com
      user: Username
      password: password

# 堡垒主机配置
# 如果集群节点需要通过堡垒机跳转，那么需要为RKE配置堡垒机信息
bastion_host:
    address: x.x.x.x
    user: ubuntu
    port: 22
    ssh_key_path: /home/user/.ssh/bastion_rsa
# or
#   ssh_key: |-
#     -----BEGIN RSA PRIVATE KEY-----
#
#     -----END RSA PRIVATE KEY-----

# Set the name of the Kubernetes cluster  

# 集群名称
cluster_name: mycluster

# 定义kubernetes版本.
# 目前, 版本定义需要与rancher/types defaults map相匹配: https://github.com/rancher/types/blob/master/apis/management.cattle.io/v3/k8s_defaults.go#L14
# 如果同时定义了kubernetes_version和system_images中的kubernetes镜像，则system_images配置将优先于kubernetes_version
kubernetes_version: v1.10.3-rancher2

# 如果没有指定system_images，那么system_images镜像默认与kubernetes_version镜像相同
# 默认Tags: https://github.com/rancher/types/blob/master/apis/management.cattle.io/v3/k8s_defaults.go)
system_images:
    kubernetes: rancher/hyperkube:v1.10.3-rancher2
    etcd: rancher/coreos-etcd:v3.1.12
    alpine: rancher/rke-tools:v0.1.9
    nginx_proxy: rancher/rke-tools:v0.1.9
    cert_downloader: rancher/rke-tools:v0.1.9
    kubernetes_services_sidecar: rancher/rke-tools:v0.1.9
    kubedns: rancher/k8s-dns-kube-dns-amd64:1.14.8
    dnsmasq: rancher/k8s-dns-dnsmasq-nanny-amd64:1.14.8
    kubedns_sidecar: rancher/k8s-dns-sidecar-amd64:1.14.8
    kubedns_autoscaler: rancher/cluster-proportional-autoscaler-amd64:1.0.0
    pod_infra_container: rancher/pause-amd64:3.1

services:
    etcd:
      # if external etcd is used
      # path: /etcdcluster
      # external_urls:
      #   - https://etcd-example.com:2379
      # ca_cert: |-
      #   -----BEGIN CERTIFICATE-----
      #   xxxxxxxxxx
      #   -----END CERTIFICATE-----
      # cert: |-
      #   -----BEGIN CERTIFICATE-----
      #   xxxxxxxxxx
      #   -----END CERTIFICATE-----
      # key: |-
      #   -----BEGIN PRIVATE KEY-----
      #   xxxxxxxxxx
      #   -----END PRIVATE KEY-----

    kube-api:
      # cluster_ip范围
      # 这必须与kube-controller中的service_cluster_ip_range匹配
      service_cluster_ip_range: 10.43.0.0/16

      # NodePort映射的端口范围
      service_node_port_range: 30000-32767
      pod_security_policy: false

      # kubernetes API server扩展参数
      # 这些参数将会替换默认值
      extra_args:
        # 启用审计日志到标准输出
        audit-log-path: "-"
        # Increase number of delete workers
        delete-collection-workers: 3
        # 将日志输出的级别设置为debug模式
        v: 4

    kube-controller:
      # Pods_ip范围
      cluster_cidr: 10.42.0.0/16
      # cluster_ip范围
      # 这必须与kube-api中的service_cluster_ip_range匹配
      service_cluster_ip_range: 10.43.0.0/16

    kubelet:
      # Base domain for the cluster
      cluster_domain: cluster.local

      # 内部DNS服务器地址
      cluster_dns_server: 10.43.0.10

      # 禁用swap
      fail_swap_on: false

      # 可以选择定义额外的卷绑定到服务
      extra_binds:
        - "/usr/libexec/kubernetes/kubelet-plugins:/usr/libexec/kubernetes/kubelet-plugins"

# 目前，只支持x509验证
# 你可以选择创建额外的SAN（主机名或IP）以添加到API服务器PKI证书。
# 如果要为control plane servers使用负载均衡器，这很有用。
authentication:
    strategy: x509
    sans:
      - "10.18.160.10"
      - "my-loadbalancer-1234567890.us-west-2.elb.amazonaws.com"

# Kubernetes认证模式
# Use `mode: rbac` 启用 RBAC
# Use `mode: none` 禁用 认证
authorization:
    mode: rbac

# 如果要设置Kubernetes云提供商，需要指定名称和配置
cloud_provider:
    name: aws

# Add-ons是通过kubernetes jobs来部署。 在超时后，RKE将放弃重试获取job状态。以秒为单位。
addon_job_timeout: 30

# 有几个网络插件可以选择，Rancher2默认flannel
network:
    plugin: flannel

# 目前只支持nginx ingress controller
# 设置`provider: none`来禁用ingress controller

ingress:
    provider: nginx

# 所有add-on都必须指定命名空间
addons: |-
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-nginx
      namespace: default
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80

addons_include:
    - https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/rook-operator.yaml
    - https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/rook-cluster.yaml
    - /path/to/manifest
```
