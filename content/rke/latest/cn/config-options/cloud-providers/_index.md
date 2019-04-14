---
title: 10 - 云提供商
weight: 10
---

RKE支持为Kubernetes集群设置特定[云提供商](https://kubernetes.io/docs/concepts/cluster-administration/cloud-providers/)功能。这些云提供商有特定的云配置，要启用云提供程序必须在集群YML中的`cloud_provider`指令下提供其名称和必要的配置选项。

* [AWS]({{< baseurl >}}/rke/latest/cn/config-options/cloud-providers/aws)
* [Azure]({{< baseurl >}}/rke/latest/cn/config-options/cloud-providers/azure)
* [OpenStack]({{< baseurl >}}/rke/latest/cn/config-options/cloud-providers/openstack)
* [vSphere]({{< baseurl >}}/rke/latest/cn/config-options/cloud-providers/vsphere)

除列表之外，RKE还支持[自定义云提供商]({{< baseurl >}}/rke/latest/cn/config-options/cloud-providers/custom)。