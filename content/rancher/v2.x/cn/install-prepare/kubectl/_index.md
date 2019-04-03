---
title: 7 - 安装和配置kubectl
weight: 7
---

`kubectl`是一个CLI命令行工具，用于运行Kubernetes集群的命令。Rancher 2.x中的许多维护和管理都需要它。

{{% accordion id="1" label="一、安装kubectl" %}}

{{% accordion id="option-a" label="1、使用包管理器安装" %}}

- Ubuntu, Debian or HypriotOS

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | \
    sudo apt-key add -
sudo touch /etc/apt/sources.list.d/kubernetes.list
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | \
    sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

- CentOS, RHEL or Fedora

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg \
    https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubectl
```

{{% /accordion %}}
{{% accordion id="option-b" label="2、Ubuntu通过snap安装" %}}

如果您使用的是Ubuntu或其他支持[snap](https://snapcraft.io/docs/core/install)包管理器的Linux发行版，则可通过`snap`安装kubectl。

1、切换到snap用户并运行安装命令:

```bash
sudo snap install kubectl --classic
```

2、测试以确保您安装的是最新的版本：

```bash
kubectl version
```

{{% /accordion %}}
{{% accordion id="option-c" label="3、macOS通过Homebrew安装" %}}

如果您使用的是macOS并使用[Homebrew](https://brew.sh/)包管理器，则可以使用`Homebrew`安装kubectl。

1、运行安装命令:

```bash
brew install kubernetes-cli
```

2、测试以确保您安装的是最新的版本:

```bash
kubectl version
```

{{% /accordion %}}
{{% accordion id="option-d" label="4、macOS通过Macports安装" %}}

如果您使用的是macOS并使用[Macports](https://macports.org/)包管理器，则可以使用Macports安装kubectl。

1、运行安装命令:

```bash
port install kubectl
```

2、测试以确保您安装的是最新的版本:

```bash
kubectl version
```

{{% /accordion %}}
{{% accordion id="option-e" label="5、Install with Powershell from PSGallery" %}}

如果您使用的是Windows并使用[Powershell Gallery](https://www.powershellgallery.com/)包管理器，则可以使用Powershell安装和更新kubectl。

- To install:

    运行安装命令(确保指定`DownloadLocation`):

    ```bash
    Install-Script -Name install-kubectl -Scope CurrentUser -Force
    install-kubectl.ps1 [-DownloadLocation <path>]
    ```
    >**Note:** If you do not specify a DownloadLocation, kubectl will be installed in the user's temp Directory.

    The installer creates $HOME/.kube and instructs it to create a config file

- To update:

    运行更新命令:

    ```sehll
    re-run Install-Script to update the installer
    re-run install-kubectl.ps1 to install latest binaries
    ```

{{% /accordion %}}
{{% accordion id="option-f" label="6、Install with Chocolatey on Windows" %}}

如果您使用的是Windows并使用[Chocolatey](https://chocolatey.org)包管理器，则可以使用Chocolatey安装kubectl。

1、运行安装命令:

```bash
choco install kubernetes-cli
```

2、测试以确保您安装的是最新的版本:

```bash
kubectl version
```

3、切换到％HOME％目录:

例如: `cd C:\users\yourusername`

4、创建`.kube`目录:

```bash
mkdir .kube
```

5、切换到创建的.kube目录:

```bash
cd .kube
```

6、配置kubectl以使用远程Kubernetes集群：New-Item config -type文件

{{% /accordion %}}
{{% accordion id="option-g" label="7、作为Google Cloud SDK的一部分" %}}

您可以将kubectl安装为Google Cloud SDK的一部分。

1、安装[Google Cloud SDK](https://cloud.google.com/sdk/).

2、运行`kubectl`安装命令:

```bash
gcloud components install kubectl
```

3、测试以确保您安装的是最新的版本:

```bash
kubectl version
```

{{% /accordion %}}
{{% accordion id="option-h" label="8、通过二进制文件安装" %}}

- MacOS

1、文件下载

通过[文件下载]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/download/)下载最新文档版本。

2、确保`kubectl`二进制文件是可执行文件。

```bash
chmod +x ./kubectl
```

3、将`kubectl`二进制文件移动到PATH路径下。

```bash
sudo mv ./kubectl /usr/local/bin/kubectl
```

- Linux

1、文件下载

通过[文件下载]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/download/)下载最新文档版本。

2、确保`kubectl`二进制文件是可执行文件。

```bash
chmod +x ./kubectl
```

3、将`kubectl`二进制文件移动到PATH路径下。

```bash
sudo mv ./kubectl /usr/local/bin/kubectl
```

- Windows

1、下载二进制文件

通过[文件下载]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/download/)下载最新文档版本。

2、将`kubectl`二进制文件移动到PATH路径下。

{{% /accordion %}}
{{% /accordion %}}
{{% accordion id="2" label="二、配置kubectl" %}}

使用RKE创建Kubernetes集群时，RKE会在本地目录中创建一个包含认证信息的配置文件`kube_config_rancher-cluster.yml`，以使用`kubectl`或`helm`等工具连接到新集群。

您可以将此文件复制到`$HOME/.kube/config`或者如果您正在使用多个Kubernetes集群，请将`KUBECONFIG`环境变量设置为路径`kube_config_rancher-cluster.yml`。

```bash
export KUBECONFIG=$(pwd)/kube_config_rancher-cluster.yml
```

测试您的连接，看看是否可以返回节点列表。

```bash
kubectl --kubeconfig=kube_configxxx.yml  get  nodes
 NAME                          STATUS    ROLES                      AGE       VERSION
165.227.114.63                Ready     controlplane,etcd,worker   11m       v1.10.1
165.227.116.167               Ready     controlplane,etcd,worker   11m       v1.10.1
165.227.127.226               Ready     controlplane,etcd,worker   11m       v1.10.1
```

{{% /accordion %}}
{{% accordion id="3" label="三、检查kubectl配置" %}}

通过获取集群状态来检查kubectl是否已正确配置：

```bash
kubectl cluster-info
```

如果您看到URL响应，则kubectl已正确配置。

如果您看到类似于以下内容的消息，则kubectl配置不正确或无法连接到Kubernetes集群。

```bash
The connection to the server <server-name:port> was \
    refused - did you specify the right host or port?
```

例如，如果您打算在笔记本电脑上（本地）运行Kubernetes集群，则需要首先安装minikube等工具，然后重新运行上述命令。

如果kubectl cluster-info返回url响应但您无法访问集群，要检查它是否配置正确，请使用：

```bash
kubectl --kubeconfig=kube_configxxx.yml cluster-info dump
```

{{% /accordion %}}
{{% accordion id="4" label="四、启用shell自动补全" %}}

kubectl包括自动补全支持，可以节省大量的输入！完成脚本本身由kubectl生成，因此您通常只需要从配置文件中调用它。这里提供了常见的例子。有关详细信息，请咨询`kubectl completion -h`。

## 1、使用bash的Linux主机

在CentOS Linux上，您可能需要安装默认情况下未安装的bash-completion软件包。

```bash
yum install bash-completion -y
```

运行`source <(kubectl completion bash)`可将kubectl自动补全添加到当前shell，要使kubectl自动补全命令自动加载:

```bash
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

## 2、使用bash的macOS主机

在macOS上，您需要先通过Homebrew安装`bash-completion`:

```bash
# If running Bash 3.2 included with macOS
brew install bash-completion
# or, if running Bash 4.1+
brew install bash-completion@2
```

Follow the "caveats" section of brew's output to add the appropriate bash completion path to your local .bashrc.

If you installed kubectl using the [Homebrew instructions](#install-with-homebrew-on-macos) then kubectl completion should start working immediately.

If you have installed kubectl manually, you need to add kubectl autocompletion to the bash-completion:

```bash
kubectl completion bash > $(brew --prefix)/etc/bash_completion.d/kubectl
```

The Homebrew project is independent from Kubernetes, so the bash-completion packages are not guaranteed to work.

## 3、Using Zsh

If you are using zsh edit the ~/.zshrc file and add the following code to enable kubectl autocompletion:

```bash
if [ $commands[kubectl] ]; then
    source <(kubectl completion zsh)
fi
```

Or when using [Oh-My-Zsh](http://ohmyz.sh/), edit the ~/.zshrc file and update the `plugins=` line to include the kubectl plugin.

```bash
plugins=(kubectl)
```

{{% /accordion %}}