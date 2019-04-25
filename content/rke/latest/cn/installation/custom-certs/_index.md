---
title: 1 - 自定义证书
weight: 1
---

*自v0.2.0起可用*

默认情况下，Kubernetes集群配置ssl证书来通信认证，RKE会自动为所有集群组件生成证书，部署Kubernetes集群后，您可以[管理这些自动生成的证书]({{< baseurl >}}/rke/latest/cn/cert-mgmt/#certificate-rotation)，您也可以使用[自定义证书]({{< baseurl >}}/rke/latest/cn/installation/custom-certs/)。

| 选项                | 描述                                                     |
| :------------------ | :------------------------------------------------------- |
| `--custom-certs`    | 使用证书目录中的自定义证书，默认目录：`/cluster_certs`。 |
| `--cert-dir`  value | 指定证书目录路径                                         |

## 一、使用自定义证书进行安装

```bash
＃使用位于默认目录`/cluster_certs`中的证书
rke up --custom-certs --config cluster.yml

＃使用位于您自己目录中的证书
rke up --custom-certs --cert-dir ~/my/own/certs --config cluster.yml
```

## 二、证书文件名称

`证书目录`中必须存在以下证书文件

|                            |                                     |                                         |          |
| :------------------------- | :---------------------------------- | :-------------------------------------- | :------- |
| Name                       | Certificate                         | Key                                     | Required |
| Master CA                  | kube-ca.pem                         | -                                       | *        |
| Kube API                   | kube-apiserver.pem                  | kube-apiserver-key.pem                  | *        |
| Kube Controller Manager    | kube-controller-manager.pem         | kube-controller-manager-key.pem         | *        |
| Kube Scheduler             | kube-scheduler.pem                  | kube-scheduler-key.pem                  | *        |
| Kube Proxy                 | kube-proxy.pem                      | kube-proxy-key.pem                      | *        |
| Kube Admin                 | kube-admin.pem                      | kube-admin-key.pem                      | *        |
| Apiserver Proxy Client     | kube-apiserver-proxy-client.pem     | kube-apiserver-proxy-client-key.pem     | *        |
| Etcd Nodes                 | kube-etcd-x-x-x-x.pem               | kube-etcd-x-x-x-x-key.pem               | *        |
| Kube Api Request Header CA | kube-apiserver-requestheader-ca.pem | kube-apiserver-requestheader-ca-key.pem |          |
| Service Account Token      | -                                   | kube-service-account-token-key.pem      |          |

## 三、生成证书签名请求（CSR）和密钥

如果要通过真实证书颁发机构（CA）创建和签署证书，可以使用RKE生成一组证书签名请求（CSR）和密钥。使用该`rke cert generate-csr`命令，您可以生成CSR和密钥。

1. 在rke配置文件`cluster.yml`中设置[节点信息]({{< baseurl >}}/rke/latest/cn/config-options/nodes/)。

1. 运行`rke cert generate-csr`以生成`cluster.yml`中的节点证书。默认情况下，CSR和密钥将保存在`./cluster_certs`。要将它们保存在不同的目录中，使用`--cert-dir`自定义保存目录。

    ```bash
    rke cert generate-csr

    INFO[0000] Generating Kubernetes cluster CSR certificates
    INFO[0000] [certificates] Generating Kubernetes API server csr
    INFO[0000] [certificates] Generating Kube Controller csr
    INFO[0000] [certificates] Generating Kube Scheduler csr
    INFO[0000] [certificates] Generating Kube Proxy csr
    INFO[0001] [certificates] Generating Node csr and key
    INFO[0001] [certificates] Generating admin csr and kubeconfig
    INFO[0001] [certificates] Generating Kubernetes API server proxy client csr
    INFO[0001] [certificates] Generating etcd-x.x.x.x csr and key
    INFO[0001] Successfully Deployed certificates at [./cluster_certs]
    ```

    **结果：**假设您通过`--cert-dir`未指定目录，则CSRs将在`./cluster_certs`目录生成。`CSR文件`文件将包含证书的正确替代DNS和IP名称，您可以使用它们通过真实CA签名机构进行证书签署。证书签名后，RKE可以将这些证书用作自定义证书。

    ```bash
    tree cluster_certs

    cluster_certs
    ├── kube-admin-csr.pem
    ├── kube-admin-key.pem
    ├── kube-apiserver-csr.pem
    ├── kube-apiserver-key.pem
    ├── kube-apiserver-proxy-client-csr.pem
    ├── kube-apiserver-proxy-client-key.pem
    ├── kube-controller-manager-csr.pem
    ├── kube-controller-manager-key.pem
    ├── kube-etcd-x-x-x-x-csr.pem
    ├── kube-etcd-x-x-x-x-key.pem
    ├── kube-node-csr.pem
    ├── kube-node-key.pem
    ├── kube-proxy-csr.pem
    ├── kube-proxy-key.pem
    ├── kube-scheduler-csr.pem
    └── kube-scheduler-key.pem

    0 directories, 16 files
    ```