---
title: 10 - 故障诊断
weight: 10
---

### Failed to set up SSH tunneling for host [xxx.xxx.xxx.xxx]: Can’t retrieve Docker Info ； Failed to dial to /var/run/docker.sock: ssh: rejected: administratively prohibited (open failed)

- 指定连接的用户无权访问Docker套接字，可以通过登录主机并运行`docker ps`命令来检查：

    ```bash
    $ ssh -i ssh_privatekey_file user@server
    user@server$ docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
    ```

    请参阅[以非root用户身份管理Docker](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user)来正确设置。

- 当使用`RedHat/CentOS`操作系统时，`root`由于[Bugzilla＃1527565](https://bugzilla.redhat.com/show_bug.cgi?id=1527565)，您无法使用用户连接到节点。您需要添加一个单独的用户并将其配置为访问Docker套接字。请参阅[以非root用户身份管理Docker](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user)来正确设置。
- SSH服务器版本不是`6.7或更高版本`，这是套接字转发工作所必需的，它用于通过SSH连接到Docker套接字。可以使用`sshd -V`连接主机来检查：

    ```bash
    $ nc xxx.xxx.xxx.xxx 22
    SSH-2.0-OpenSSH_6.6.1p1 Ubuntu-2ubuntu2.10
    ```

### Failed to dial ssh using address [xxx.xxx.xxx.xxx:xx]: Error configuring SSH: ssh: no key found

- 指定的`ssh_key_path`密钥文件无法访问，确保您指定了私钥文件（而不是公钥`.pub`），并且运行该`rke`命令的用户可以访问私钥文件。
- 指定的`ssh_key_path`密钥文件格式错误，通过运行`ssh-keygen -y -e -f private_key_file`检查私钥是否有效。这将打印私钥的公钥，如果私钥文件无效，将会失败。

#### 无法使用地址[xxx.xxx.xxx.xxx：xxx]拨打ssh：ssh：握手失败：ssh：无法进行身份验证，尝试的方法[none publickey]，不支持的方法仍然存在

- 指定的密钥文件`ssh_key_path`对于访问节点不正确。仔细检查是否`ssh_key_path`为节点指定了正确的，以及是否指定了要连接的正确用户。

### Failed to dial ssh using address [xxx.xxx.xxx.xxx:xx]: ssh: handshake failed: ssh: unable to authenticate, attempted methods [none publickey], no supported methods remain

- 如果您想使用加密的私钥，您应该使用`ssh-agent`用您的口令加载密钥。您可以通过在命令行上指定`--ssh-agent-auth`来配置RKE来使用该代理，它将在运行RKE命令的环境中使用`SSH_AUTH_SOCK`环境变量。

### Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

- 通过节点配置的`address`和`port`无法访问该节点