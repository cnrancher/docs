---
title: 5 - 系统镜像
weight: 5
---

从`v0.1.6`之前，当RKE部署Kubernetes时，会提取几个镜像，这些镜像用作Kubernetes系统组件。从`v0.1.6`开始，几个系统镜像的功能合并到一个`rancher/rke-tools`镜像中，以简化和加快部署过程。

您可以分别配置[network plug-ins]({{< baseurl >}}/rke/latest/cn/config-options/add-ons/network-plugins/), [ingress controller]({{< baseurl >}}/rke/latest/cn/config-options/add-ons/ingress-controllers/)和[dns provider]({{< baseurl >}}/rke/latest/cn/config-options/add-ons/dns/)以及这些插件的设置选项。这是用于通过RKE部署Kubernetes的完整系统镜像列表的示例，镜像版本依赖于所使用的[Kubernetes image/version](https://github.com/rancher/types/blob/master/apis/management.cattle.io/v3/k8s_defaults.go)

> **Note:** 随着RKE版本的发布，这些镜像上的标签将不再是最新的。此列表是特定`v1.10.3-rancher2`版本。

```yaml
system_images:
  etcd: rancher/coreos-etcd:v3.2.24
  alpine: rancher/rke-tools:v0.1.24
  nginx_proxy: rancher/rke-tools:v0.1.24
  cert_downloader: rancher/rke-tools:v0.1.24
  kubernetes: rancher/hyperkube:v1.13.1-rancher1
  kubernetes_services_sidecar: rancher/rke-tools:v0.1.24
  pod_infra_container: rancher/pause-amd64:3.1

  # kube-dns images
  kubedns: rancher/k8s-dns-kube-dns-amd64:1.15.0
  dnsmasq: rancher/k8s-dns-dnsmasq-nanny-amd64:1.15.0
  kubedns_sidecar: rancher/k8s-dns-sidecar-amd64:1.15.0
  kubedns_autoscaler: rancher/cluster-proportional-autoscaler-amd64:1.0.0

  # CoreDNS images
  coredns: coredns/coredns:1.2.6
  coredns_autoscaler: rancher/cluster-proportional-autoscaler-amd64:1.0.0

  # Flannel images
  flannel: rancher/coreos-flannel:v0.10.0
  flannel_cni: rancher/coreos-flannel-cni:v0.3.0

  # Calico images
  calico_node: rancher/calico-node:v3.4.0
  calico_cni: rancher/calico-cni:v3.4.0
  calico_controllers: ""
  calico_ctl: rancher/calico-ctl:v2.0.0

  # Canal images
  canal_node: rancher/calico-node:v3.4.0
  canal_cni: rancher/calico-cni:v3.4.0
  canal_flannel: rancher/coreos-flannel:v0.10.0

  # Weave images
  weave_node: weaveworks/weave-kube:2.5.0
  weave_cni: weaveworks/weave-npc:2.5.0

  # Ingress controller images
  ingress: rancher/nginx-ingress-controller:0.21.0-rancher1
  ingress_backend: rancher/nginx-ingress-controller-defaultbackend:1.4

  # Metrics server image
  metrics_server: rancher/metrics-server-amd64:v0.3.1
```

在`v0.1.6`之前，我们使用以下镜像代替`rancher/rke-tools`镜像：

```yaml
system_images:
    alpine: alpine:latest
    nginx_proxy: rancher/rke-nginx-proxy:v0.1.1
    cert_downloader: rancher/rke-cert-deployer:v0.1.1
    kubernetes_services_sidecar: rancher/rke-service-sidekick:v0.1.0
```