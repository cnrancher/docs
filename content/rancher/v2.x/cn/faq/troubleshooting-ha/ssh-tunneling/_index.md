---
title: 4、无法为主机创建SSH隧道
weight: 4
---

## Failed to set up SSH tunneling for host [xxx.xxx.xxx.xxx]: Can't retrieve Docker Info ，Failed to dial to /var/run/docker.sock: ssh: rejected: administratively prohibited (open failed)

- 指定连接的用户没有权限访问docker.sock。这可以通过登录主机并运行`docker ps`命令来检查 :

```
$ ssh user@server
user@server$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
```

- 当使用`RedHat/CentOS`作为操作系统时, 不能使用`root`用户去登录主机，具体原因可以查看[Bugzilla #1527565](https://bugzilla.redhat.com/show_bug.cgi?id=1527565)。需要添加一个非root用户并添加访问docker.sock的权限，配置方法请参考[通过非root用户管理Docker](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user)。

- SSH server version is not version 6.7 or higher. This is needed for socket forwarding to work, which is used to connect to the Docker socket over SSH. This can be checked using `sshd -V` on the host you are connecting to, or using netcat:

```
$ nc xxx.xxx.xxx.xxx 22
SSH-2.0-OpenSSH_6.6.1p1 Ubuntu-2ubuntu2.10
```

## Failed to dial ssh using address [xxx.xxx.xxx.xxx:xx]: Error configuring SSH: ssh: no key found

- 在RKE的`node`配置参数中，`ssh_key_path`参数需要指定访问`node`节点的私钥文件。此问题可能是因为没有正确指定`ssh_key_path`文件路径或者没有权限访问该文件，或者指定的文件非正确的私钥文件。

## Failed to dial ssh using address [xxx.xxx.xxx.xxx:xx]: ssh: handshake failed: ssh: unable to authenticate, attempted methods [none publickey], no supported methods remain

- 指定的`ssh_key_path`文件对应的`node`主机不正确，或者对应的用户名不正确。

## Failed to dial ssh using address [xxx.xxx.xxx.xxx:xx]: Error configuring SSH: ssh: cannot decode encrypted private keys

- If you want to use encrypted private keys, you should use `ssh-agent` to load your keys with your passphrase. If the `SSH_AUTH_SOCK` environment variable is found in the environment where the `rke` command is run, it will be used automatically to connect to the node.

## Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

- 无法通过配置的`address`和`port`访问到主机，检查主机防火墙或者配置的`address`和`port`。
