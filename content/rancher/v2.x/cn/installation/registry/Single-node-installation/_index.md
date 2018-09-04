---
title: 1 - Harbor单节点安装
weight: 1
---
本文档翻译至Harbor官方文档[Installation and Configuration Guide](https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md)

{{% accordion id="option-a" label="一、环境准备" %}}

## 1、硬件

| 资源 | 配置      | 描述       |
| -------- | ------------- | ----------------- |
| CPU      | 最少 2 CPU | 4CPU(推荐)             |
| 内存      | 最少 4GB   | 8GB(推荐)          |
| 磁盘      | *          | *             |

>**注意:** 磁盘大小需要根据实际使用场景来确定

## 2、软件

| 软件名       | 版本                 | 描述                                                  |
| -------------- | ----------------------- | ------------------------------------------------------------ |
| Python         | 2.7或更高版本   | **注意:** 你可能必须在Linux发行版(Gentoo，Arch)上安装Python，默认情况下没有安装Python解释器 |
| Docker         | 1.10或更高版本  | 有关安装说明，请参考: <https://docs.docker.com/engine/installation/> |
| Docker-Compose | 1.6.0或更高版本 | 有关安装说明，请参考: <https://docs.docker.com/compose/install/> |
| Openssl        | 推荐最新版本     | 为Harbor生成证书和密钥                    |

## 3、网络

| 端口 | 协议 | 描述                                                  |
| ---- | -------- | ------------------------------------------------------------ |
| 443  | HTTPS    | Harbor UI和API将接受此端口上的https协议请求 |
| 4443 | HTTPS    | 只有在启用`Notary`时才需要连接到`Docker Content Trust`服务 |
| 80   | HTTP     | Harbor UI和API将接受此端口上的http协议请求 |

{{% /accordion %}}
{{% accordion id="option-b" label="二、Harbor配置" %}}

## 1、Harbor程序下载

可以通过Harbor的[发布页面](https://github.com/goharbor/harbor/releases)下载，也可以通过[文件下载](/docs/rancher/v2.x/cn/installation/download/)页面下载最新的在线安装包。

## 2、解压压缩包

`tar xvf harbor-online-installer-<version>.tgz`

## 3、修改配置文件

解压压缩包会得到harbor文件夹，harbor.cfg配置文件位于文件夹根目录。在harbor.cfg中有两类参数，必需参数和可选参数。

- **必要参数**:需要在配置文件中设置这些参数。如果用户更新harbor.cfg并运行install.sh脚本以重新安装Harbor，更改的参数将生效。

- **可选参数**:这些参数对于更新是可选的，即用户可以将它们保留为默认值，并在启动Harbour后在Web UI上更新它们。如果已经配置harbor.cfg，这些参数只会在首次启动Harbour时生效。harbor.cfg将忽略对这些参数的后续修改。

>**注意**:如果你选择通过UI设置这些参数，请务必在Harbour启动后立即执行此操作。特别是，你必须在Harbour中注册或创建新用户之前，设置所需的auth_mode。当系统中有用户时(除默认管理员用户外)，无法更改auth_mode。

参数如下所述 - 请注意，至少需要更改`hostname`参数。

### 必需参数

- **hostname**:目标主机的主机名，用于访问UI和Harbor服务。它应该是目标计算机的IP地址或域名(FQDN)，例如，`192.168.1.10`或`reg.yourdomain.com`。不要使用localhost或127.0.0.1作为主机名，因为外部客户端需要访问Harbor服务！

- **ui_url_protocol** :( **http**或**https**。默认为**http**)用于访问UI和令牌/通知服务的协议。如果启用了公证，则此参数必须为*https*。默认情况下，这是*http*。要设置https协议，请参阅**使用HTTPS访问配置Harbor**。

- **db_password**:用于**db_auth**的MySQL数据库的root密码。`生产环境请修改此密码`

- **max_job_workers** :(默认值为**3**)作业服务中的最大复制工作数。对于每个镜像复制作业，程序将存储库的所有标记同步到远程目标。增加此数量可以在系统中实现更多并发复制作业。但是，由于每个复制进程都消耗一定的网络/CPU/IO资源，`请根据主机的硬件资源仔细选择该参数的值`。

- **customize_crt**:(**on**或**off**，默认为**on**)，如果此属性**on**，准备脚本创建私钥和根证书，用于生成/验证`registry`的令牌。当外部源提供密钥和根证书时，将此属性设置为**off**。有关详细信息，请参阅[自定义密钥和harbor令牌服务证书](https://github.com/goharbor/harbor/blob/master/docs/customize_token_service.md)。

- **ssl_cert**:SSL证书的路径，仅在协议设置为https时应用

- **ssl_cert_key**:SSL密钥的路径，仅在协议设置为https时应用

- **secretkey_path**:用于加密或解密复制策略中远程Harbor密码的密钥路径。

- **log_rotate_count**:日志文件在被删除之前会被轮转**log_rotate_count**次。如果count为0，则删除旧版本而不是轮转。

- **log_rotate_size**:仅当日志文件大于**log_rotate_size**字节时才会轮转日志文件。如果大小后跟k，则假定大小以千字节为单位。如果使用M，则大小以兆字节为单位，如果使用G，则大小为千兆字节。尺寸100，尺寸100k，尺寸100M和尺寸100G都是有效的。

### 可选参数

- **Email settings**:Harbor需要这些参数才能向用户发送“密码重置”电子邮件，并且仅在需要该功能时才做配置。另外，**请注意**，在默认情况下SSL连接`没有启用`，如果你的SMTP服务器需要SSL，那么你应该通过设置`email_ssl = TRUE`参数来启用SSL，但`不支持STARTTLS`。如果电子邮件服务器使用自签名证书或不受信任证书，则需要设置`email_insecure = true`。有关`email_identity`的详细说明，请参阅[rfc2595](https://tools.ietf.org/rfc/rfc2595.txt)

  ```bash
    - email_server = smtp.mydomain.com
    - email_server_port = 25
    - email_identity =
    - email_username = [sample_admin@mydomain.com](mailto:sample_admin@mydomain.com)
    - email_password = abc
    - email_from = admin [sample_admin@mydomain.com(mailto:sample_admin@mydomain.com)
    - email_ssl = false
    - email_insecure = false
  ```

- **harbor_admin_password**:管理员的初始密码。此密码仅在Harbor首次启动时生效。之后将忽略此设置，并且应在UI中设置管理员密码。**请注意**:默认用户名/密码为`admin/Harbor12345`

- **auth_mode**:使用的身份验证类型。默认情况下，它是**db_auth**，即凭据存储在数据库中。对于LDAP身份验证，请将其设置为**ldap_auth**。

    **重要信息:**从现有Harbor实例升级时，必须确保在启动新版本的Harbor之前,`harbor.cfg`配置文件中**auth_mode**相同。否则，用户可能无法在升级后登录。

- **ldap_url**:LDAP连接URL(例如`ldaps://ldap.mydomain.com`)。 `仅在**auth_mode**设置为ldap_auth时使用`。

- **ldap_searchdn**:具有搜索LDAP/AD服务器权限的用户的DN(例如`uid=admin,ou=people,dc=mydomain,dc=com`)。

- **ldap_search_pwd**:**ldap_searchdn**指定的用户密码。

- **ldap_basedn**:查找用户的基本DN，例如`ou=people,dc=mydomain,dc=com`。 `仅在**auth_mode**设置为ldap_auth时使用。`

- **ldap_filter**:用于查找用户的搜索过滤器，例如`(objectClass=person)`。

- **ldap_uid**:用于在LDAP搜索期间匹配用户的属性，它可以是uid，cn，email或其他属性。

- **ldap_scope**:搜索用户的范围，0-LDAP_SCOPE_BASE，1-LDAP_SCOPE_ONELEVEL，2-LDAP_SCOPE_SUBTREE。默认值为2。

- **self_registration** :( **on**或**off**。默认**on**)启用/禁用用户自助注册功能。禁用时，新用户只能由管理员用户创建，只有管理员可以在Harbor中创建新用户。 ***注意:当**auth_mode**设置为**ldap_auth时**，**始终**禁用自助注册功能，并忽略此设置。*

- **token_expiration**:令牌服务创建的令牌到期时间(以分钟为单位)，默认为30分钟。

- **project_creation_restriction**:用于控制用户有权创建项目的设置。默认情况下，每个人都可以创建一个项目，设置为“adminonly”，只有管理员才能创建项目。

### 配置存储后端(可选)

默认情况下，Harbor将镜像存储在本地文件系统中。在生产环境中，你可以考虑使用其他存储后端而不是本地文件系统，如S3，OpenStack Swift，Ceph等。你需要更新的是storage文件中的部分common/templates/registry/config.yml。例如，如果你使用Openstack Swift作为存储后端，则该部分可能如下所示:

```bash
storage:
  swift:
    username: admin
    password: ADMIN_PASS
    authurl: http://keystone_addr:35357/v3/auth
    tenant: admin
    domain: default
    region: regionOne
    container: docker_images
```

**注意**:有关注册表存储后端的详细信息，请参阅[registry配置参考](https://docs.docker.com/registry/configuration/)。

### Harbor监听自定义端口

默认情况下，Harbor监听80(HTTP)和443(HTTPS，如果已配置)，你可以使用自定义命令对其进行修改。

- 对于HTTP协议

  1、修改 docker-compose.yml

  将第一个`80`修改为自定义端口，例如`8888:80`。

  ```yaml
  proxy:
      image: library/nginx:1.11.5
      restart: always
      volumes:
        - ./config/nginx:/etc/nginx
      ports:
        - 8888:80
        - 443:443
      depends_on:
        - mysql
        - registry
        - ui
        - log
      logging:
        driver: "syslog"
        options:  
          syslog-address: "tcp://127.0.0.1:1514"
          tag: "proxy"
  ```

  2、修改harbor.cfg，将端口添加到参数“hostname”

  `hostname = 192.168.0.2:8888`

  3、重新部署Harbour,参考上一节“管理harbor的生命周期”。

- 对于HTTPS协议

  方法同`HTTP协议`

{{% /accordion %}}
{{% accordion id="option-c" label="三、Harbor安装" %}}

在harbor文件夹中有`install.sh`脚本，一旦`harbor.cfg`和存储后端(可选)配置完成，就可以镜像Harbor安装。请注意，在线安装需要一些时间从Docker hub下载Harbor镜像，具体根据实际网络情况。

- 默认安装(没有Notary/Clair)

  `sudo ./install.sh`

  >Harbor已与Notary和Clair集成(用于漏洞扫描)。但是，默认不安装Notary或Clair服务。

  如果一切正常，你应该能够打开浏览器访问`http://reg.yourdomain.com/`上的管理门户(`reg.yourdomain.com`为`harbor.cfg`配置的主机名,默认管理员用户名/密码为admin/Harbor12345)。

  登录管理门户并创建一个新项目，例如: myproject。然后，你可以使用docker命令登录和推送镜像。默认情况下，Harbor的默认安装使用HTTP协议，而Docker默认信任https协议。所以，要想docker命令登录和推送镜像，需要添加`--insecure-registry`到docker 配置文件并重启docker服务。

  ```bash
  docker login reg.yourdomain.com
  docker push reg.yourdomain.com/myproject/myrepo:mytag
  ```

- 使用Notary安装

  要使用Notary服务安装Harbour，请在运行`install.sh`时添加参数:

  `sudo ./install.sh --with-notary`

  **注意**:使用notary安装，参数ui_url_protocol必须设置为“https”。

- 使用Clair安装

  要使用Clair服务安装Harbour，请在运行`install.sh`时添加参数:

  `sudo ./install.sh --with-clair`

- 同时安装Clair和Notary

  `sudo ./install.sh --with-notary --with-clair`

{{% /accordion %}}
{{% accordion id="option-d" label="四、配置HTTPS访问配置Harbor" %}}

Harbor不附带任何证书，默认情况下使用HTTP来处理请求。虽然这使得设置和运行相对简单 - 特别是对于开发或测试环境 - 但不建议用于生产环境。要启用HTTPS，请参阅[使用HTTPS访问Harbor](/docs/rancher/v2.x/cn/installation/registry/configure-https/)。

{{% /accordion %}}
{{% accordion id="option-e" label="五、Harbor的生命周期" %}}

## 1、默认安装管理Harbor的生命周期

- 开始、停止、重启

  你可以使用docker-compose来管理Harbor的生命周期。一些有用的命令列出如下(必须与docker-compose.yml在同一目录中运行)。

  `sudo docker-compose start/stop/restart`

- 更新配置

  要更改Harbour的配置，请先停止现有的Harbor实例并进行更新harbor.cfg。然后运行prepare脚本以填充配置。最后重新创建并启动Harbor的实例:

  ```bash
  sudo docker-compose down -v
  sudo vim harbor.cfg
  sudo prepare
  sudo docker-compose up -d
  ```

- 删除Harbor的容器，同时将镜像数据和Harbor的数据库文件保存在文件系统上

  `sudo docker-compose down -v`

- 删除Harbor的数据库和图像数据(用于干净的重新安装)

  ```bash
  rm -r /data/database
  rm -r /data/registry
  ```

## 2、与notary或者Clair一起安装时管理Harbor的生命周期

当Harbour与Notary或者Clair一起安装时，docker-compose命令需要指定一个或者两个额外的模板文件。用于管理Harbour生命周期的docker-compose命令是:

```bash
sudo docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml [ up|down|ps|stop|start ]
```

```bash
sudo docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml -f ./docker-compose.clair.yml [ up|down|ps|stop|start ]
```

- 如果要在使用Notary安装Harbor时更改配置并重新部署Harbour，则应使用以下命令:

  ```bash
  sudo docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml down -v
  sudo vim harbor.cfg
  sudo prepare --with-notary
  sudo docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml up -d
  ```

  ```bash
  sudo docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml -f   ./docker-compose.clair.yml down -v
  sudo vim harbor.cfg
  sudo prepare --with-notary --with-clair
  sudo docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml -f   ./docker-compose.clair.yml up -d
  ```

{{% /accordion %}}
{{% accordion id="option-f" label="六、持久数据和日志文件" %}}

默认情况下，镜像数据保留在主机的/data/目录中。即使Harbor的容器被移除或重新创建，此数据仍保持不变。此外，Harbor使用rsyslog来收集每个容器的日志。默认情况下，这些日志文件存储在目标主机上的`/var/log/harbor/`目录中以进行故障排除。

{{% /accordion %}}
{{% accordion id="option-g" label="七、性能调整" %}}

默认情况下，Harbor将Clair容器的CPU使用率限制为150000，并避免耗尽所有CPU资源。这在docker-compose.clair.yml文件中定义,你可以根据硬件配置对其进行修改。

{{% /accordion %}}
{{% accordion id="option-h" label="八、故障排除" %}}

- 当Harbor无法正常工作时，请运行以下命令以查明Harbor的所有容器是否处于UP状态:

  `sudo docker-compose ps`

  如果容器不是UP状态，检查目录容器的日志文件/var/log/harbor。例如，如果容器harbor-ui未运行，则应查看日志文件ui.log。

{{% /accordion %}}
