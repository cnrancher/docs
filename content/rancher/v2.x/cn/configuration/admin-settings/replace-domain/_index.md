---
title: 10 - 变更Racnher server域名
weight: 10
---

## **重要提示**

- 在master节点的/etc/kubernetes/.tmp路径下找到kubecfg-kube-admin.yaml

  ![image-20190309173154246](assets/image-20190309173154246.png)

- 操作之前备份Rancher Server

  备份方法参考：{{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/

## 一、准备证书

更换域名后，会涉及到证书无法验证的问题,所以更换证书也一般会涉及到证书的更换。

- 自签名ssl证书

可通过以下脚本自动创建以域名为名称的自签名ssl证书。

```bash
#!/bin/bash -e

# *为必改项

# * 新域名
DOMANI=''

# 扩展信任IP(一般ssl证书只信任域名的访问请求，有时候需要使用ip去访问server，那么需要给ssl证书添加扩展IP)
IP='' # 示例写法：'IP:172.16.155.35, IP:172.16.155.36, DNS:www.rancher.com'

# 国家名(2个字母的代号)
C=CN
# 省
ST=rancher
# 市
L=rancher
# 公司名
O=rancher
# 组织或部门名
OU=rancher
# 服务器FQDN或颁发者名(更换为你自己的域名)
CN=${DOMANI}
# 证书有效期
DATE=${DATE:-3650}
# 邮箱地址
EMAILADDRESS=support@rancher.com

echo "1. 生成CA证书"
openssl req -newkey rsa:4096 -nodes -sha256 -keyout cakey.pem -x509 -days ${DATE} -out cacerts.pem \
-subj "/C=${C}/ST=${ST}/L=${L}/O=${O}/OU=${OU}/CN=ca.${CN}/emailAddress=${EMAILADDRESS}"

echo "2. 生成证书签名请求文件"
openssl req -newkey rsa:4096 -nodes -sha256 -keyout ${CN}.key -out  ${CN}.csr \
-subj "/C=${C}/ST=${ST}/L=${L}/O=${O}/OU=${OU}/CN=${CN}/emailAddress=${EMAILADDRESS}"

echo "3. 创建服务证书"
echo "3.1. 添加扩展IP"
echo "subjectAltName = ${IP}" > extfile.cnf
echo "3.2. 生成服务证书"
openssl x509 -req -days ${DATE} -in ${CN}.csr -CA cacerts.pem -CAkey cakey.pem -CAcreateserial -extfile extfile.cnf -out  ${CN}.crt

echo "4. 重命名服务证书"
cp ${CN}.key tls.key
cp ${CN}.crt tls.crt

```
- 权威认证证书

把证书重命名为需要的文件名称

```bash
cp xxx.key tls.key
cp xxx.crt tls.crt
```

## 二、更新证书

### 1、Rancher独立容器运行

>注意：操作前先备份，{{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/backups/

如果是以容器映射证书文件运行的rancher server，只需要停止原有rancher server容器，然后使用相同的运行配置，替换新证书后重新运行新容器即可，注意容器名称不要相同。

### 2、Rancher HA运行

>注意：操作前先备份，{{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/backups/

- 保存原有证书yaml文件

```bash
kubectl -n cattle-system get secret tls-rancher-ingress  -o yaml > tls-ingress.yaml
kubectl -n cattle-system get secret tls-ca  -o yaml > tls-ca.yaml
```

- 删除旧的secret，然后创建新的secret

```bash
# 指定kube配置文件路径

kubeconfig=

# 删除旧的secret
kubectl --kubeconfig=$kubeconfig -n cattle-system delete secret tls-rancher-ingress
kubectl --kubeconfig=$kubeconfig -n cattle-system delete secret tls-ca

# 创建新的secret
kubectl --kubeconfig=$kubeconfig -n cattle-system create secret tls tls-rancher-ingress --cert=./tls.crt --key=./tls.key
kubectl --kubeconfig=$kubeconfig -n cattle-system create secret generic tls-ca --from-file=cacerts.pem

# 重启Pod
kubectl  --kubeconfig=$kubeconfig -n cattle-system delete pod `kubectl --kubeconfig=$kubeconfig -n cattle-system get pod |grep -E "cattle-cluster-agent|cattle-node-agent|rancher" | awk '{print $1}'`
```

>**重要提示：**如果环境不是按照标准的rancher安装文档安装，secret名称可能不相同，请根据实际secret名称操作。

## 三、更新local集群的密文`cattle-credentials-xxxx`

1. 查看集群API ![image](assets/67C9FB42B207498F9B63A82A824F2E36.png)
2. 点击**clusterRegistrationTokens** ![image-20190309184052331](_index.assets/image-20190309184052331.png)
3. 此处找到新的TOKEN 和CA-CHECKSUM，记下来备用![image-20190309184523316](_index.assets/image-20190309184523316.png)

4. 在`system`项目\资源\密文 路径页面下，找到并编辑密文`cattle-credentials-xxxx`

![image](assets/F95130C0C6414121B351515EA538B70C.png)

- url字段修改为新的域名
- token字段修改为查找到的新token

## 四、修改local集群agent的域名和CA-CHECKSUM

- 在`system`项目下找到cattle-system命名空间，分别编辑cattle-cluster-agent和cattle-node-agent![image-20190309185550237](_index.assets/image-20190309185550237.png)

- 修改`CATTLE_SERVER`为新的域名，检查`CA-CHECKSUM`是否有变化，如果有则需要一并修改。![image](assets/796DBF17B0FB46148E2AE31D39D6DA26.png)

## 五、修改业务集群agent pod

因为token和域名的更改，业务集群的agent pod无法连接rancher server,这个时候需要通过命令行工具去编辑yaml文件。

- 执行以下命令：

```bash
kubeconfig=''
kubectl --kubeconfig=$kubeconfig -n cattle-system edit deployments cattle-cluster-agent
kubectl --kubeconfig=$kubeconfig -n cattle-system edit daemonsets cattle-node-agent
```

- 然后编辑环境变量

![image-20190309190814057](_index.assets/image-20190309190814057.png)