---
title: 主机模板
weight: 3
---

配置节点池集群时，将使用主机模板来配置集群节点。这些模板使用Docker Machine配置选项来定义节点的操作系统映像和设置参数。创建主机模板时，它将绑定到您的用户配置文件，主机模板不能在用户之间共享，您可以删除不再使用的陈旧主机模板。

## 创建主机模板

1. 选择用户头像>主机模板
2. 单击添加模板
3. 选择一个可用的云提供商，然后按照屏幕上的说明配置模板

**Result:** The template is configured. You can use the template later when you [provision a node pool cluster]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/rke-clusters/node-pools).

## 克隆主机模板

创建新主机模板时，可以克隆现有模板并快速更新其设置，而不是从头开始创建新模板。克隆模板可以为你节省配置时间，避免重新输入云提供商访问密钥的麻烦。

1. 选择用户头像>主机模板
2. 找到要克隆的模板。然后选择省略号>克隆
3. 填写表格的其余部分

**Result:** The template is cloned and configured. You can use the template later when you [provision a node pool cluster]({{< baseurl >}}/rancher/v2.x/en/cluster-provisioning/rke-clusters/node-pools).

## 删除主机模板

当你不再使用主机模板时，可以从用户设置中删除它。

1. 选择用户头像>主机模板。
2. 从列表中选择一个或多个模板，然后单击删除，出现提示时确认删除。