---
title: 10 - 系统工具
weight: 10
---

## 一：下载 System Tools

您可以从 [GitHub releases page](https://github.com/rancher/system-tools/releases/latest)根据操作系统类型下载最新版本的 `system-tools` 。

| 操作系统 | 文件名                           |
| :--------------- | :------------------------------- |
| MacOS            | `system-tools_darwin-amd64`      |
| Linux            | `system-tools_linux-amd64`       |
| Windows          | `system-tools_windows-amd64.exe` |

下载工具后，完成以下操作:

1. 将文件重命名为 `system-tools`.

2. 通过以下命令给与文件可执行权限:

    > **使用Windows?** 该文件已经是可执行文件，您可以跳过此步骤。

    ```bash
    chmod +x system-tools
    ```

## 二：使用 System Tools

以下子命令可用:

| 命令              | 描述                               |
| :---------------- | :--------------------------------- |
| [logs](#logs)     | 从节点收集Kubernetes集群组件日志。 |
| [stats](#stats)   | 主机状态指标                       |
| [remove](#remove) | 删除Rancher创建的Kubernetes资源。  |

### 1、Logs

系统工具将使用提供的kubeconfig配置文件部署守护进程，该守护进程将从核心Kubernetes集群组件复制所有日志文件，并将它们添加到一个`tar`文件中 (默认名称：`cluster-logs.tar` ). 如果只想从单个节点收集日志记录，可以使用 `--node NODENAME` 或者 `-n NODENAME`指定节点。

- 用法

    ```bash
    ./system-tools_darwin-amd64 logs --kubeconfig <KUBECONFIG>
    ```

- 参数

    | 参数                                                   | 描述                                                         |
    | :----------------------------------------------------- | :----------------------------------------------------------- |
    | `--kubeconfig <KUBECONFIG_PATH>, -c <KUBECONFIG_PATH>` | 集群kubeconfig文件。                                         |
    | `--output <FILENAME>, -o cluster-logs.tar`             | 创建的日志文件`tar`文件名称，如果没有指定文件名，默认为 `cluster-logs.tar`. |
    | `--node <NODENAME>, -n node1`                          | 指定要从中收集日志的节点，如果没有指定节点，则将收集集群中所有节点的日志。 |

### 2、Stats

stats子命令将显示集群节点的系统指标

系统工具将部署一个守护进程，并运行一个预定义命令`sar` (System Activity Report)来显示系统指标。

- 用法

    ```bash
    ./system-tools_darwin-amd64 stats --kubeconfig <KUBECONFIG>
    ```

- 参数

    | 参数                                                   | 描述                                                         |
    | :----------------------------------------------------- | :----------------------------------------------------------- |
    | `--kubeconfig <KUBECONFIG_PATH>, -c <KUBECONFIG_PATH>` | 集群kubeconfig文件。                                         |
    | `--node <NODENAME>, -n node1`                          | 指定要从中收集日志的节点，如果没有指定节点，则将收集集群中所有节点的日志。 |
    | `--stats-command value, -s value`                      | 运行显示系统指标的命令，如果没有指定命令，默认为 `/usr/bin/sar -u -r -F 1 1`. |

### 3、Remove

当在Kubernetes集群上安装Rancher时，它将创建Kubernetes资源来运行和存储配置数据。如果希望从集群中删除Rancher，可以使用`remove`子命令删除Kubernetes资源。当使用remove子命令时，将删除以下资源:

> **警告:** 这个命令将从etcd节点中删除数据，在执行命令之前，确保已进行[数据备份]({{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/backups/)

- Rancher部署命名空间 (默认为：`cattle-system` ).
- `serviceAccount`, `clusterRoles`, 和 `clusterRoleBindings` 都会设置 `cattle.io/creator:norman`标签， Rancher从v2.1.0开始创建的任何资源都配置`cattle.io/creator:norman`标签。
- Labels, annotations, 和 finalizers。
- Rancher Deployment。
- Machines, clusters, projects, 和用户自定义部署资源 (CRDs)。
- 所有在 `management.cattle.io` API组下创建的资源。
- 所有由Rancher v2.x创建的CRDs资源。

    > **使用v2.0.8或更早版本?**
    >
    > v2.0.8或更早版本不会在job运行后自动删除`serviceAccount、clusterRole和clusterRoleBindings`资源,需要手动删除它们。

- 用法

运行下面的命令时，将从集群中删除[Remove](#Remove)列出的所有资源

> **警告:** 这个命令将从etcd节点中删除数据，在执行命令之前，确保已进行[数据备份]({{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/backups/)

    ```bash
    ./system-tools remove --kubeconfig <KUBECONFIG> --namespace <NAMESPACE>
    ```

- 参数

    | 参数                                                   | 描述                                                         |
    | :----------------------------------------------------- | :----------------------------------------------------------- |
    | `--kubeconfig <KUBECONFIG_PATH>, -c <KUBECONFIG_PATH>` | 集群kubeconfig文件。                                         |
    | `--namespace <NAMESPACE>, -n cattle-system`            | Rancher 2.x deployment 命名空间 (`<NAMESPACE>`). 如果没有指定命名空间，默认为：  `cattle-system`. |
    | `--force`                                              | 跳过交互式删除确认操作，并在没有提示的情况下删Rancher 。     |