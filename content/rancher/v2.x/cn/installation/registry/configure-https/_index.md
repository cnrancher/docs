---
title: 3 - Harbor配置HTTPS
weight: 3
---
{{% accordion id="option-a" label="一、获得证书" %}}

证书可以选择权威机构颁发的证书，也可以通过[自签名ssl](/docs/rancher/v2.x/cn/installation/self-signed-ssl/)生成需要的自签名ssl证书。

{{% /accordion %}}
{{% accordion id="option-b" label="二、配置和安装" %}}

1、获取`yourdomain.com.crt和yourdomain.com.key`文件后，将它们放入以下目录`/root/cert/`

```
cp yourdomain.com.crt /root/cert/
  cp yourdomain.com.key /root/cert/
```

2、接下来，编辑`harbor.cfg`文件，更新主机名和协议，并设置`ssl_cert`和`ssl_cert_key`路径

```
#set hostname
  hostname = reg.yourdomain.com
  #set ui_url_protocol
  ui_url_protocol = https
  ......
  #The path of cert and key files for nginx, they are applied only the protocol is set to https 
  ssl_cert = /root/cert/yourdomain.com.crt
  ssl_cert_key = /root/cert/yourdomain.com.key
```

3、如果Harbor已在运行，请停止并删除现有实例，你的镜像数据保留在文件系统中。如果没运行，则跳过此步骤。

`docker-compose down -v`

4、重新生成配置

`./prepare`

5、重新运行实例

`docker-compose up -d`

{{% /accordion %}}
{{% accordion id="option-c" label="三、验证" %}}

为Harbor设置HTTPS后，通过以下步骤进行验证:

1、打开浏览器并输入地址:https//reg.yourdomain.com。它应该显示Harbor的用户界面。

2、在安装了Docker的计算机上，请确保`-insecure-registry`配置不存在，并且将上述步骤中生成的`ca.crt`复制到`/etc/docker/certs.d/reg.yourdomain.com`(或Harbor主机IP)，如果该目录不存在，则创建它。

如果你将nginx443端口映射到另一个端口，那么你应该创建`/etc/docker/certs.d/reg.yourdomain.com:port`(或Harbor主机IP:port)。然后运行docker命令来验证设置，例如:

`docker login reg.yourdomain.com`

如果你已将nginx 443端口映射到另一个端口，则需要将端口添加到登录命令中，如下所示:

`docker login reg.yourdomain.com:port`

{{% /accordion %}}