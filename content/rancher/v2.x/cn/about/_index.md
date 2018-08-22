---
title: 关于
weight: 10
---

## 代码库

所有的代码库都位于我们的GitHub组织内，Rancher使用了许多代码库，我们将对Rancher使用的一些主要代码库进行描述:

Repository | URL | Description
-----------|-----|-------------
Rancher | https://github.com/rancher/rancher | 存储Rancher 2.x的主要源代码
Types | https://github.com/rancher/types | 存储Rancher 2.x的所有API类型
API Framework | https://github.com/rancher/norman | 存储构建由Kubernetes Custom Resources支持的Rancher样式API的API框架
User Interface | https://github.com/rancher/ui | 存储Rancher UI源代码
(Rancher) Docker Machine | https://github.com/rancher/machine | 存储`Docker Machine`二进制文件的源代码。这是`docker/machine`仓库的一个分支
machine-package | https://github.com/rancher/machine-package | 构建Rancher Docker Machine二进制文件的源代码
kontainer-engine | https://github.com/rancher/kontainer-engine | 存储`kontainer-engine`的源代码
RKE repository | https://github.com/rancher/rke | 存储Rancher Kubernetes Engine源代码
CLI | https://github.com/rancher/cli | 存储Rancher CLI的源代码
(Rancher) Helm repository | https://github.com/rancher/helm | Helm二进制文件的源代码。这是`helm/helm`仓库的一个分支
Telemetry repository | https://github.com/rancher/telemetry | `telemetry`二进制文件的源代码
loglevel repository | https://github.com/rancher/loglevel | 储库loglevel二进制文件的源代码，用于动态更改日志级别

要查看所有Rancher中使用的`libraries/projects`，请参考`rancher/rancher`仓库中`vendor.conf`的内容。

## 构建

每个代码库都应该有一个Makefile文件，可以使用该make命令构建。`make`执行的目标程序是基于存储库中`/scripts`目录中的脚本(以及其他`trash`命令，有关使用`trash`的更多信息），每个目标将使用`Dapper`在隔离环境中运行。`Dockerfile.dapper`将用于此过程，并包含所需的所有必要的构建工具。

默认运行的目标程序是`ci`,并且将会运行`./scripts/validate`，`./scripts/build`，`./scripts/test`和`./scripts/package`。生成的二进制文件将包含在`./build/binDocker`镜像中，并且通常也打包在Docker镜像中。

使用`trash`管理对其他`libraries/projects`的依赖性，查看`trash README`了解它如何使用。简而言之，它使用一个`vendor.conf`文件来指定源存储库和获取的x版本，签出和复制到`./vendor`目录。更新`vendor.conf`后，您可以运行`make trash`更新依赖项。更新依赖项后，您可以再次使用`make`构建项目，以便使用更新的依赖项来构建。

## Bugs, Issues or Questions

如果您发现任何错误或有任何问题，请搜索[Rancher issues](https://github.com/rancher/rancher/issues)，因为有人可能遇到过同样的问题，或者我们正在积极研究解决方案。

如果您找不到与您的问题相关的任何内容，请通过[提交issues](https://github.com/rancher/rancher/issues/new)与我们联系。虽然我们有许多与Rancher相关的代码库，但我们希望在`rancher/rancher`代码库中提交issues，避免我们错过它！如果您想询问问题或向其他用户交流使用方法或者技术方案，我们建议您在[Rancher论坛](https://forums.cnrancher.com)上创建一个帖子。

在提交问题时请遵循此表，这将有助于我们调查和解决问题。更多信息意味着我们可以使用更多数据来确定导致问题的原因或可能与问题相关的内容。

>**注意:** 对于大量数据，请使用[GitHub Gist](https://gist.github.com/)或类似数据存储工具，并在问题中附加创建的链接资源。
>
>**重要提示:** 请删除所有敏感数据，因为它们是公开可见的。

- 尽可能详细地提供所使用资源的详细信息。由于问题的原因可能很多，尽可能详细的细节有助于确定具体原因。看下面的一些例子：
  - 主机(what cloud does it happen on, what Amazon Machine Image are you using, what DigitalOcean droplet are you using, what image are you provisioning that we can rebuild or use when we try to reproduce)
  - 操作系统(What operating system are you using. Providing specifics helps here like the output of `cat /etc/os-release` for exact OS release and `uname -r` for exact kernel used)
  - Docker (What Docker version are you using, how did you install it? Most of the details of Docker can be found by supplying output of `docker version` and `docker info`)
  - 环境(Are you in a proxy environment, are you using recognized CA/self signed certificates, are you using an external loadbalancer)
  - Rancher(What version of Rancher are you using, this can be found on the bottom left of the UI or be retrieved from the image tag you are running on the host)
  - 集群(What kind of cluster did you create, how did you create it, what did you specify when you were creating it)
- 提供手动步骤或自动化脚本，用于重新复现问题
- 从使用的资源中提供`数据/日志`
  - Rancher

    - `Single node`

    ```bash
    docker logs \
    --tail=all \
    --timestamps \
    $(docker ps  -q -f label=org.label-schema.vcs-url=https://github.com/rancher/rancher.git)
    ```
    - `High Availability`

    ```bash
    kubectl --kubeconfig $KUBECONFIG logs \
    -n cattle-system \
    --timestamps=true \
    -f $(kubectl --kubeconfig $KUBECONFIG get pods -n cattle-system -o json | jq -r '.items[] | select(.spec.containers[].name="cattle-server") | .metadata.name')
    ```
  - System logging(这些可能并非全部存在，具体取决于操作系统)

    - `/var/log/messages`
    - `/var/log/syslog`
    - `/var/log/kern.log`

  - Docker daemon logging(这些可能并非全部存在，具体取决于操作系统)
    - Ubuntu (old using upstart ) - `/var/log/upstart/docker.log`
    - Ubuntu (new using systemd ) - `sudo journalctl -fu docker.service`
    - Boot2Docker - `/var/log/docker.log`
    - Debian GNU/Linux - `/var/log/daemon.log`
    - CentOS - `cat /var/log/daemon.log | grep docker`
    - CoreOS - `journalctl -u docker.service`
    - Fedora - `journalctl -u docker.service`
    - Red Hat Enterprise Linux Server - `cat /var/log/messages | grep docker`
    - OpenSuSE - `journalctl -u docker.service`
    - OSX - `~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/log/d‌​ocker.log`
    - Windows - `Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time`

如果您遇到性能问题，请提供尽可能多的指标数据(文件或屏幕截图)。如果您有关于机器的问题，提供`top、free -hm、df -h、iostat`的输出信息，这表明进程/内存/磁盘空间/磁盘IO的使用情况。

## 文档

- [Rancher 2.x Docs Repo](https://github.com/rancher/docs): Rancher 2.x文档仓库，它们位于`content`文件夹中。

- [Rancher 1.x Docs Repo](https://github.com/rancher/rancher.github.io): Rancher 1.x文档仓库，它们位于`rancher`文件夹中。
