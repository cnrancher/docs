---
title: 4 - 堡垒/跳板主配置
weight: 4
---

由于RKE使用`ssh`连接到节点，所以可以配置为使用堡垒/跳板主机。

```yaml
bastion_host:
    address: x.x.x.x
    user: ubuntu
    port: 22
    ssh_key_path: /home/user/.ssh/bastion_rsa
    # or
    # ssh_key: |-
    #   -----BEGIN RSA PRIVATE KEY-----
    #
    #   -----END RSA PRIVATE KEY-----
    # Optionally using SSH certificates
    # ssh_cert_path: /home/user/.ssh/id_rsa-cert.pub
    # or
    # ssh_cert: |-
    #   ssh-rsa-cert-v01@openssh.com AAAAHHNza...
```

## 堡垒主机选项

### Adddress

该`address`参数用于设置堡垒主机的主机名或IP地址,RKE必须能够连接到此地址。

### SSH Port

指定连接到堡垒主机时要使用的节点port。默认端口是22。

### SSH Users

指定连接到堡垒主机时要使用的节点user。

### SSH Key Path

`ssh_key_path`,指定连接到堡垒主机时要使用的SSH私钥路径，每个节点的默认密钥路径是~/.ssh/id_rsa。

### SSH Key

相对`ssh_key_path`，`ssh_key`可以设置实际密钥内容，而不是设置SSH密钥文件的路径。

### SSH Certificate Path

连接到节点时要使用的已签名SSH证书，可以通过`ssh_cert_path`指定ssh_cert路径。

### SSH Certificate

相对`ssh_cert_path`，`ssh_cert`可以指定实际证书内容，而不是设置签名SSH证书的路径。
