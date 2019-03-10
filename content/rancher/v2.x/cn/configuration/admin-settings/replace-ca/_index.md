---
title: 9 - 修改k8s证书后Rancher端需要做的操作
weight: 9
---


## Rancher相关备份操作

### 备份yaml

- 在集群层级，点击`项目/命名空间`导出所有的命名空间yaml文件；
- 在各个项目级别，分别导出工作负载、负载均衡、服务发现、数据卷、配置映射、密文的yaml文件；

### 备份etcd（HA部署）

在存有RKE配置文件的目录执行以下命令:

```
./rke_linux-amd64 etcd snapshot-save --name SNAPSHOT-201903xx.db --config cluster.yml
```
RKE会获取每个etcd节点的快照，并保存在每个etcd节点的`/opt/rke/etcd-snapshots`目录下。

## 进行业务集群的k8s证书升级

证书过期一般是由于`ca.crt`过期，验证有效期的方法如下

```
openssl x509 -in ca.crt -noout -dates
```
CA证书文件一般路径在`/etc/kubernetes/ssl`，如果找不到，可以在`local`集群中执行如下命令获取

```
kubectl get clusters <CLUSTER_ID> -o jsonpath={.status.caCert} | base64 -d
```
>CLUSTER_ID，集群ID可在浏览器地址栏查看。

### 示例

```
rancher-server1-root@rancher-server1:~# kubectl get clusters c-2xqt7 -o 

jsonpath={.status.caCert} | base64 -d
-----BEGIN CERTIFICATE-----
MIIFnTCCA4WgAwIBAgIJAPnVRHuosezjMA0GCSqGSIb3DQEBCwUAMGUxCzAJBgNV
BAYTAkNOMREwDwYDVQQIDAhTaGFuZ2hhaTEPMA0GA1UEBwwGUHVkb25nMRYwFAYD
VQQKDA1zYWljbW90b3IuY29tMQwwCgYDVQQLDANrOHMxDDAKBgNVBAMMA2s4czAe
Fw0xODAzMDgwNjI2NDNaFw0xOTAzMDgwNjI2NDNaMGUxCzAJBgNVBAYTAkNOMREw
DwYDVQQIDAhTaGFuZ2hhaTEPMA0GA1UEBwwGUHVkb25nMRYwFAYDVQQKDA1zYWlj
bW90b3IuY29tMQwwCgYDVQQLDANrOHMxDDAKBgNVBAMMA2s4czCCAiIwDQYJKoZI
hvcNAQEBBQADggIPADCCAgoCggIBAK/e4muA4zUtBG7WtXC7J46jdAxZmYmh6W/j
zTpzir8yKcvocx/ELPZ6qVdSXZJPb5IKC1RAJz/xM2pvU5oAzgRc8inxz3AFHBZM
1oHVoIq5pVG0XVBt6o1N16myrHaCgwPLnytQ5lhJD09safmIGSShTktB6JdhA/7y
1ol8NLrFnzBH4/8oNwCChy+qGXTOa566S8c5noCg6NZ/LlYV1KPyIv1LnTfwulzU
zzRLe0VDcezpRf4FrxMRKK83VEIJvRqnTxE3hYAH4AyvShpQseIHEtA4sJyRK69H
lWE+FE2ozY3jhpVftl3zJj/7S2oHxBZQtBSizIOmoj7kTm0cxS1zqjdz6k/NAhCW
dUwQOmEW/N2sxZ/9A3KtGxgERJsm1EFHcVC3oqcVemnt08TIpChbvJ9nphDUwgEN
ikxod8W8012Cvir71AGcwfsQsa2NpdqBuP8m0AZN0HX0RflNsTMyUhP6z72u1Crw
xfohyowh0LO3heL9/aHwwGTD8hN0n6dbjPNE/YMDwEFiO5dqtQjh2heerHFr6oPP
facHBNc7YJi3/ANJMrVzTs5EadX6XY+INRWNNTlKL/lkzwogtvLX0V40RfISWkyK
CRcrK/bqjFHKYTqIWOvPvtY7LhT/K0tjTMV1dALsJ59yLIT28vX/YtmcadhFlt6P
LdMwyEwrAgMBAAGjUDBOMB0GA1UdDgQWBBTSG2dcHpgXJHTONTgeQtQWargg4TAf
BgNVHSMEGDAWgBTSG2dcHpgXJHTONTgeQtQWargg4TAMBgNVHRMEBTADAQH/MA0G
CSqGSIb3DQEBCwUAA4ICAQAoUvv1cX0cPVzWh6q1gJGd2AND7w6WfXNbumdxi3PR
MiaTvvE59Ud7hSJ5MLUcSI56HS+hdfF8U1/919E5I2m1LhcbqU8hFtn6ZGJr+P5z
4iCyXpvgvJeXqauqgvOZlBlc1Xm6YhyOyC98bk9fJtiTePIVOzv3edWmZIBnu/MT
VbObqIkzIpaHCjZh7wfnhU3uLzujFt4oZNQcIQsOYaLg1QHW/rFvKm01sog7odo/
ZEhz71YkhtMblmkY7LoUR9ql6LaxuYglTis+zczJdB4xpR/29WTkVVJ29ETnxS55
QMNnZJsAtfubc9B51x0r/LWzSH8iBmlnpjiY2SyJEvWqJj8rJCAFfgXDJt37rffz
BxLl9xFor4maWLY7zpGu/7sMz0CYdYyTeQvIss9/ChqTSEqQmtcdaKNdJiyrdBhF
ZQXhR21YHdxrsorrIC4qbq7eWnCjpzzBCEHpqL+FwGwbvXdMNZNJTo1RaKOMXdfA
XXqcYqZWltrserDKasxZ30OYMaH2RwNjvZAqQRVauliiF8feJlHt9JZHT45dPCaO
Bc973rbUJ/xZYxUpONaCl9vV5ZDoD4y2TSx5C1iM/U1aNKd5FWww+E7GZpLiCFx4
HAR5R84cBjbGLG9qLWa+TTLQeZrHuq1NTmbDCttiJVwFM7bii5eTPsJAauRFS744
7Q==
-----END CERTIFICATE-----
```

升级完业务集群的证书之后，Rancher中集群会报x509错误，需要执行以下步骤修复。


## 修复步骤

需要准备的文件

- `ca.crt` 新的ca证书文件
- `kubeconfig` 更新证书后的集群链接配置文件

>注: 假设kubeconfig文件位置
- 老证书对应的文件位置:`207:/home/appuser/rke/new_dev/kubeconfig-new-dev-rancher`
- 新证书对应的文件位置:`207:/home/appuser/cacrt/config-test2-new`


### 1. 在业务集群中删除`token`

删除其中的`cattle-token-xxx`，其会基于最新证书配置启动重建

```
rancher-test-appuser@rancher-test:~/cacrt$ kubectl --kubeconfig=config-test2-new -n cattle-system get secret
NAME                         TYPE                                  DATA      AGE
cattle-credentials-096434b   Opaque                                2         61d
cattle-credentials-152dd17   Opaque                                2         61d
cattle-token-2vff8           kubernetes.io/service-account-token   3         1d
default-token-r2wtl          kubernetes.io/service-account-token   3         61d
```
生成新`token`之后，取值解密备用。

```
kubectl -n cattle-system get secret cattle-token-9n9f5 -o jsonpath={.data.token} | base64 -d 
```

### 2. 在local集群中编辑业务集群的crd clusters对象做二个变更

- 备份集群crd对象

```
kubectl get clusters <cluster-id> -o yaml > <cluster-id>.yaml
```

- `caCert`字段参数，用新的ca证书文件base64加密后替换。

直接cat完整的证书文件，然后base64加密。

```
cat ca.crt | base64
```
- `serviceAccountToken`字段的参数，使用步骤1中的token解密后替换。


