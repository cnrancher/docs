---
title: 10 - 变更Racnher server域名
weight: 10
---

## **重要提示**

- 2.1.x以前的版本，可在master节点的/etc/kubernetes/.tmp路径下找到kubecfg-kube-admin.yml

  ![image-20190309173154246](assets/image-20190309173154246.png)

- 操作之前备份Rancher Server

  备份方法参考：{{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/

## 一、准备证书

更换域名后，会涉及到证书无法验证的问题,所以更换证书也一般会涉及到证书的更换。

- 自签名ssl证书

复制以下代码另存为`create_self-signed-cert.sh`或者其他您喜欢的文件名。修改代码开头的`CN`(域名)，如果需要使用ip去访问Rancher Server，那么需要给ssl证书添加扩展IP，多个IP用逗号隔开。如果想实现多个域名访问Rancher Server，则添加扩展域名(SSL_DNS),多个`SSL_DNS`用逗号隔开。

```bash
#!/bin/bash -e

help ()
{
    echo  ' ================================================================ '
    echo  ' --ssl-domain: 生成ssl证书需要的主域名，如不指定则默认为localhost，如果是ip访问服务，则可忽略；'
    echo  ' --ssl-trusted-ip: 一般ssl证书只信任域名的访问请求，有时候需要使用ip去访问server，那么需要给ssl证书添加扩展IP，多个IP用逗号隔开；'
    echo  ' --ssl-trusted-domain: 如果想多个域名访问，则添加扩展域名（SSL_TRUSTED_DOMAIN）,多个扩展域名用逗号隔开；'
    echo  ' --ssl-size: ssl加密位数，默认2048；'
    echo  ' --ssl-date: ssl有效期，默认10年；'
    echo  ' --ca-date: ca有效期，默认10年；'
    echo  ' --ssl-cn: 国家代码(2个字母的代号),默认CN;'
    echo  ' 使用示例:'
    echo  ' ./create_self-signed-cert.sh --ssl-domain=www.test.com --ssl-trusted-domain=www.test2.com \ '
    echo  ' --ssl-trusted-ip=1.1.1.1,2.2.2.2,3.3.3.3 --ssl-size=2048 --ssl-date=3650'
    echo  ' ================================================================'
}

case "$1" in
    -h|--help) help; exit;;
esac

if [[ $1 == '' ]];then
    help;
    exit;
fi

CMDOPTS="$*"
for OPTS in $CMDOPTS;
do
    key=$(echo ${OPTS} | awk -F"=" '{print $1}' )
    value=$(echo ${OPTS} | awk -F"=" '{print $2}' )
    case "$key" in
        --ssl-domain) SSL_DOMAIN=$value ;;
        --ssl-trusted-ip) SSL_TRUSTED_IP=$value ;;
        --ssl-trusted-domain) SSL_TRUSTED_DOMAIN=$value ;;
        --ssl-size) SSL_SIZE=$value ;;
        --ssl-date) SSL_DATE=$value ;;
        --ca-date) CA_DATE=$value ;;
        --ssl-cn) CN=$value ;;
    esac
done

# CA相关配置
CA_DATE=${CA_DATE:-3650}
CA_KEY=${CA_KEY:-cakey.pem}
CA_CERT=${CA_CERT:-cacerts.pem}
CA_DOMAIN=localhost

# ssl相关配置
SSL_CONFIG=${SSL_CONFIG:-$PWD/openssl.cnf}
SSL_DOMAIN=${SSL_DOMAIN:-localhost}
SSL_DATE=${SSL_DATE:-3650}
SSL_SIZE=${SSL_SIZE:-2048}

## 国家代码(2个字母的代号),默认CN;
CN=${CN:-CN}

SSL_KEY=$SSL_DOMAIN.key
SSL_CSR=$SSL_DOMAIN.csr
SSL_CERT=$SSL_DOMAIN.crt

echo -e "\033[32m ---------------------------- \033[0m"
echo -e "\033[32m       | 生成 SSL Cert |       \033[0m"
echo -e "\033[32m ---------------------------- \033[0m"

if [[ -e ./${CA_KEY} ]]; then
    echo -e "\033[32m ====> 1. 发现已存在CA私钥，备份"${CA_KEY}"为"${CA_KEY}"-bak，然后重新创建 \033[0m"
    mv ${CA_KEY} "${CA_KEY}"-bak
    openssl genrsa -out ${CA_KEY} ${SSL_SIZE}
else
    echo -e "\033[32m ====> 1. 生成新的CA私钥 ${CA_KEY} \033[0m"
    openssl genrsa -out ${CA_KEY} ${SSL_SIZE}
fi

if [[ -e ./${CA_CERT} ]]; then
    echo -e "\033[32m ====> 2. 发现已存在CA证书，先备份"${CA_CERT}"为"${CA_CERT}"-bak，然后重新创建 \033[0m"
    mv ${CA_CERT} "${CA_CERT}"-bak
    openssl req -x509 -sha256 -new -nodes -key ${CA_KEY} -days ${CA_DATE} -out ${CA_CERT} -subj "/C=${CN}/CN=${CA_DOMAIN}"
else
    echo -e "\033[32m ====> 2. 生成新的CA证书 ${CA_CERT} \033[0m"
    openssl req -x509 -sha256 -new -nodes -key ${CA_KEY} -days ${CA_DATE} -out ${CA_CERT} -subj "/C=${CN}/CN=${CA_DOMAIN}"
fi

echo -e "\033[32m ====> 3. 生成Openssl配置文件 ${SSL_CONFIG} \033[0m"
cat > ${SSL_CONFIG} <<EOM
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, serverAuth
EOM

if [[ -n ${SSL_TRUSTED_IP} || -n ${SSL_TRUSTED_DOMAIN} ]]; then
    cat >> ${SSL_CONFIG} <<EOM
subjectAltName = @alt_names
[alt_names]
EOM
    IFS=","
    dns=(${SSL_TRUSTED_DOMAIN})
    dns+=(${SSL_DOMAIN})
    for i in "${!dns[@]}"; do
      echo DNS.$((i+1)) = ${dns[$i]} >> ${SSL_CONFIG}
    done

    if [[ -n ${SSL_TRUSTED_IP} ]]; then
        ip=(${SSL_TRUSTED_IP})
        for i in "${!ip[@]}"; do
          echo IP.$((i+1)) = ${ip[$i]} >> ${SSL_CONFIG}
        done
    fi
fi

echo -e "\033[32m ====> 4. 生成服务SSL KEY ${SSL_KEY} \033[0m"
openssl genrsa -out ${SSL_KEY} ${SSL_SIZE}

echo -e "\033[32m ====> 5. 生成服务SSL CSR ${SSL_CSR} \033[0m"
openssl req -sha256 -new -key ${SSL_KEY} -out ${SSL_CSR} -subj "/C=${CN}/CN=${SSL_DOMAIN}" -config ${SSL_CONFIG}

echo -e "\033[32m ====> 6. 生成服务SSL CERT ${SSL_CERT} \033[0m"
openssl x509 -sha256 -req -in ${SSL_CSR} -CA ${CA_CERT} \
    -CAkey ${CA_KEY} -CAcreateserial -out ${SSL_CERT} \
    -days ${SSL_DATE} -extensions v3_req \
    -extfile ${SSL_CONFIG}

echo -e "\033[32m ====> 7. 证书制作完成 \033[0m"
echo
echo -e "\033[32m ====> 8. 以YAML格式输出结果 \033[0m"
echo "----------------------------------------------------------"
echo "ca_key: |"
cat $CA_KEY | sed 's/^/  /'
echo
echo "ca_cert: |"
cat $CA_CERT | sed 's/^/  /'
echo
echo "ssl_key: |"
cat $SSL_KEY | sed 's/^/  /'
echo
echo "ssl_csr: |"
cat $SSL_CSR | sed 's/^/  /'
echo
echo "ssl_cert: |"
cat $SSL_CERT | sed 's/^/  /'
echo

echo -e "\033[32m ====> 9. 附加CA证书到Cert文件 \033[0m"
cat ${CA_CERT} >> ${SSL_CERT}
echo "ssl_cert: |"
cat $SSL_CERT | sed 's/^/  /'
echo

echo -e "\033[32m ====> 10. 重命名服务证书 \033[0m"
echo "cp ${SSL_DOMAIN}.key tls.key"
cp ${SSL_DOMAIN}.key tls.key
echo "cp ${SSL_DOMAIN}.crt tls.crt"
cp ${SSL_DOMAIN}.crt tls.crt

```

- 权威认证证书

把证书重命名为需要的文件名称

```bash
cp xxx.key tls.key
cp xxx.crt tls.crt
```

## 二、更新证书

### 1、Rancher单节点运行

>注意：操作前先备份，{{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/backups/

如果是以容器映射证书文件运行的Rancher Server，只需要停止原有Rancher Server容器，然后使用相同的运行配置，替换新证书后重新运行新容器即可，注意容器名称不要相同。

### 2、Rancher HA运行

>注意：操作前先备份，{{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/backups/

- 保存原有证书yaml文件

```bash
kubectl --kubeconfig=kube_configxxx.yml -n cattle-system \
get secret tls-rancher-ingress  -o yaml > tls-ingress.yml

kubectl --kubeconfig=kube_configxxx.yml -n cattle-system \
get secret tls-ca  -o yaml > tls-ca.yml
```

- 删除旧的secret，然后创建新的secret

```bash
# 指定kube配置文件路径

kubeconfig=

# 删除旧的secret
kubectl --kubeconfig=$kubeconfig -n cattle-system \
delete secret tls-rancher-ingress

kubectl --kubeconfig=$kubeconfig -n cattle-system \
delete secret tls-ca

# 创建新的secret
kubectl --kubeconfig=$kubeconfig -n cattle-system \
create secret tls tls-rancher-ingress --cert=./tls.crt --key=./tls.key

kubectl --kubeconfig=$kubeconfig -n cattle-system \
create secret generic tls-ca --from-file=cacerts.pem

# 重启Pod
kubectl --kubeconfig=$kubeconfig -n cattle-system \
delete pod `kubectl --kubeconfig=$kubeconfig -n cattle-system \
get pod |grep -E "cattle-cluster-agent|cattle-node-agent|rancher" | awk '{print $1}'`
```

>**重要提示：**如果环境不是按照标准的rancher安装文档安装，`secret`名称可能不相同，请根据实际secret名称操作。

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

因为token和域名的更改，业务集群的agent pod无法连接Rancher Server,这个时候需要通过命令行工具去编辑yaml文件。

- 执行以下命令：

```bash
kubeconfig=''
kubectl --kubeconfig=$kubeconfig -n cattle-system \
edit deployments cattle-cluster-agent

kubectl --kubeconfig=$kubeconfig -n cattle-system \
edit daemonsets cattle-node-agent
```

- 然后编辑环境变量

![image-20190309190814057](_index.assets/image-20190309190814057.png)