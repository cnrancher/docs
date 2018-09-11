---
title: FAQ(持续更新)
weight: 8
---

## 一、常见问题

### 1、何为PEM格式？

PEM格式通常用于数字证书认证机构(Certificate Authorities，CA)，扩展名为.pem, .crt, .cer, and .key。内容为Base64编码的ASCII码文件，有类似"-----BEGIN CERTIFICATE-----" 和 "-----END CERTIFICATE-----"的头尾标记。服务器认证证书，中级认证证书和私钥都可以储存为PEM格式(认证证书其实就是公钥)。Apache和类似的服务器使用PEM格式证书。

你可以通过以下特征识别PEM格式:

  ```
  - 该文件以下列标题开头:
  -----BEGIN CERTIFICATE-----
  - 标题后面跟着一串长字符
  - 该文件以页脚结尾:
  -----END CERTIFICATE-----
  ```

**PEM证书例如:**

  ```
  ----BEGIN CERTIFICATE-----
  MIIGVDCCBDygAwIBAgIJAMiIrEm29kRLMA0GCSqGSIb3DQEBCwUAMHkxCzAJBgNV
  ... more lines
  VWQqljhfacYPgp8KJUJENQ9h5hZ2nSCrI+W00Jcw4QcEdCI8HL5wmg==
  -----END CERTIFICATE-----
  ```

### 2、如果我想添加我的中间证书，证书的顺序是什么？

添加证书的顺序如下:

```
-----BEGIN CERTIFICATE-----
%YOUR_CERTIFICATE%
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
%YOUR_INTERMEDIATE_CERTIFICATE%
-----END CERTIFICATE-----
```

### 3、我如何验证我的证书链？

你可以使用`openssl`二进制验证证书链。如果该命令的输出(参见下面的命令示例)结束`Verify return code: 0 (ok)`，那么证书链是有效的。该`ca.pem`文件必须与你添加到`rancher/rancher`容器中的文件相同。当使用由认可的认证机构签署的证书时，可以省略该`-CAfile`参数。

**命令:**

```
openssl s_client -CAfile ca.pem -connect rancher.yourdomain.com:443
...
Verify return code: 0 (ok)
```

### 4、持久数据

Rancher `etcd`用作数据存储，使用单节点安装时，将使用内置`etcd`。持久数据位于容器中的以下路径中: `/var/lib/rancher`。你可以将主机卷挂载到此位置以保留其运行的数据。

**命令**:

```
# 指定主机路径
  HOST_PATH=xxxx
  docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v $HOST_PATH:/var/lib/rancher \
  rancher/rancher:latest
```

### 5、如何在同一个主机上运行`Rancher/Rancher`和`Rancher/Rancher-Agent`

在你想要使用单个节点运行Rancher并且能够将相同节点添加到集群的情况下，你必须调整为`rancher/rancher`容器映射的主机端口。

如果一个节点被添加到集群，它将部署使用端口80和443的ingress控制器。这与`rancher/rancher`容器默认映射的端口冲突。

>**注意**不建议在生产中把Rancher/Rancher和Rancher/Rancher-Agent运行在一台主机上，但可用于开发/演示。

要更改主机端口映射，替换`-p 80:80 -p 443:443`为`-p 8080:80 -p 8443:443`:

```
docker run -d --restart=unless-stopped \
  -p 8080:80 -p 8443:443 \
  rancher/rancher:latest
```

### Kubernetes

#### What does it mean when you say Rancher v2.0 is built on Kubernetes?

Rancher v2.0 is a complete container management platform built on 100% on Kubernetes leveraging its Custom Resource and Controller framework.  All features are written as a CustomResourceDefinition (CRD) which extends the existing Kubernetes API and can leverage native features such as RBAC.

#### Do you plan to implement upstream Kubernetes, or continue to work on your own fork?

We're still going to provide our distribution when you select the default option of having us create your Kubernetes cluster, but it will be very close to upstream. 

#### Does this release mean that we need to re-train our support staff in Kubernetes?

Yes.  Rancher will offer the native Kubernetes functionality via `kubectl` but will also offer our own UI dashboard to allow you to deploy Kubernetes workload without having to understand the full complexity of Kubernetes.  However, to fully leverage Kubernetes, we do recommend understanding Kubernetes.  We do plan on improving our UX with subsequent releases to make Kubernetes easier to use.

#### So, wait. Is a Rancher compose going to make a Kubernetes pod? Do we have to learn both now? We usually use the filesystem layer of files, not the UI.

No.  Unfortunately, the differences were enough such that we cannot support Rancher compose anymore in 2.0.  We will be providing both a tool and guides to help with this migration.

### Cattle

### How does Rancher v2.0 affect Cattle?

Cattle will not supported in v2.0 as Rancher has been re-architected to be based on Kubernetes. You can, however, expect majority of Cattle features you use will exist and function similarly on Kubernetes. We will develop migration tools in Rancher v2.1 to help you transform your existing Rancher Compose files into Kubernetes YAML files.

#### Can I migrate existing Cattle workloads into Kubernetes?

Yes. In the upcoming Rancher v2.1 release we will provide a tool to help translate existing Cattle workloads in Compose format to Kubernetes YAML format.  You will then be able to deploy those workloads on the v2.0 platform.

### 环境和集群

#### 我还可以为环境和集群创建模板吗？

不可以. 从2.0开始，环境的概念已经改为Kubernetes集群，并且只支持Kubernetes调度引擎。

#### Can you still add an existing host to an environment? (i.e. not provisioned directly from Rancher)

Yes. We still provide you with the same way of executing our Rancher agents directly on hosts.

### 升级和迁移

#### 如何从v1.x迁移到v2.0？

由于将Docker容器转换为Kubernetes pod的技术难度，升级将要求用户将这些工作负载从v1.x迁移到新的v2.0环境中。我们计划在v2.1中增加一个工具，将现有的Rancher Compose文件转换为Kubernetes YAML文件。然后，你将能够在v2.0平台上部署这些工作负载。

#### Is it possible to upgrade from Rancher v1.0 to v2.0 without any disruption to Cattle and Kubernetes clusters?

At this time, we are still exploring this scenario and taking feedback. We anticipate that you will need to launch a new Rancher instance and then relaunch on v2.0. Once you've moved to v2.0, upgrades will be in place, as they are in v1.6.

#### Can I import OpenShift Kubernetes clusters into v2.0?

Our goal is to run any upstream Kubernetes clusters. Therefore, Rancher v2.0 should work with OpenShift, but we haven't tested it yet.

### Support

#### What about Rancher v1.6? Are you planning some long-term support releases?

That is definitely the focus of the v1.6 stream. We're continuing to improve that release, fix bugs, and maintain it for the next 12 months at a minimum. We will extend that time period, if necessary, depending on how quickly users move to v2.1.

#### Does Rancher v2.0 support Docker Swarm and Mesos as environment types?

When creating an environment in Rancher v2.0, Swarm and Mesos will no longer be standard options you can select. However, both Swarm and Mesos will continue to be available as Catalog applications you can deploy. It was a tough decision to make but, in the end, it came down to adoption. For example, out of more than 15,000 clusters, only about 200 or so are running Swarm.

#### Is it possible to manage Azure Container Services with Rancher v2.0?
Yes.

#### What about Windows support?

We plan to provide Windows support for v2.1 based on Microsoft’s new approach to providing an overlay network using Kubernetes and CNI. This new approach matches well with what we are doing in v2.1 and, once that is complete, you will be able to leverage the same Rancher UX, or Kubernetes UX, but with Windows. We are in the middle of discussing how we can make this happen with Microsoft, and we will provide more information before the end of this year.

#### Are you planning on supporting Istio in Rancher v2.0?

We like Istio, and it's something we're looking at potentially integrating and supporting.

#### Will Rancher v2.0 support Hashicorp's Vault for storing secrets?

Not yet. We currently support Hashicorp's Vault in v1.6 and plan on supporting it in an upcoming release post v2.0.

#### Does Rancher v2.0 support RKT containers as well?

At this time, we only support Docker.

#### Will Rancher v2.0 support Calico, Contiv, Contrail, Flannel, Weave net, etc., for embedded and imported Kubernetes?

We will initially only support Calico, Canal, and Flannel.

#### Are you planning on supporting Traefik for existing setups?

We don't currently plan on providing embedded Traefik support, but we're still exploring load-balancing approaches.

### General

#### Can we still add our own infrastructure services, which had a separate view/filter in 1.6.x?

Yes. We plan to eventually enhance this feature so you can manage Kubernetes storage, networking, and its vast ecosystem of add-ons.

#### Are you going to integrate Longhorn?

Yes. Longhorn was on a bit of a hiatus while we were working on v2.0. We plan to re-engage on the project once v2.0 reaches GA (general availability).

#### Are there changes to default roles available now or going forward? Will the Kubernetes alignment impact plans for roles/RBAC?

The default roles will be expanded to accommodate the new Rancher 2.0 features, and will also take advantage of the Kubernetes RBAC (Role-Based Access Control) capabilities to give you more flexibility.

#### Will there be any functions like network policies to separate a front-end container from a back-end container through some kind of firewall in v2.0?

Yes. You can do so by leveraging Kubernetes' network policies.

#### What about the CLI? Will that work the same way with the same features?

Yes. Definitely.

#### If we use Kubernetes native YAML files for creating resources, should we expect that to work as expected, or do we need to use Rancher/Docker compose files to deploy infrastructure?

Absolutely.
