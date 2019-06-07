---
title: 1 - API审计日志
weight: 1
---

Rancher附带API审计功能，用以记录每个用户发起的系统事件信息。您可以知道发生了什么、何时发生、谁发起的、以及事件对集群的影响。API审计也记录Rancher API的所有请求和响应，包括使用Rancher UI和通过编程使用Rancher API的所有其他用途。

您可以在Rancher安装或升级期间启用API审计。

## 一、API审计日志设置选项

以下定义了有关审计日志记录的内容以及包含哪些数据的规则：

参数 | 描述
---------|----------
 `AUDIT_LEVEL` | `0` - 禁用审计日志(默认设置)<br/>`1` - 记录事件元数据<br/>`2` - 记录事件元数据和请求内容. <br/>`3` - 记录事件元数据、请求内容和响应内容。请求/响应对的每个日志事务使用相同的`auditID`值。<br/>有关显示每个等级设置记录的具体内容，请参阅[审计级别日志](#二-审计日志级别)记录
 `AUDIT_LOG_PATH` | Rancher Server API的日志路径，默认路径是：`/var/log/auditlog/rancher-api-audit.log`，您可以将日志目录挂载到主机。<br/>用法示例: `AUDIT_LOG_PATH=/my/custom/path/`<br/> 
 `AUDIT_LOG_MAXAGE` | 定义了保留旧审计日志文件的最大天数，默认为10天。
 `AUDIT_LOG_MAXBACKUP` | 定义要保留的最大审计日志文件个数，默认值为10。
 `AUDIT_LOG_MAXSIZE` | 定义单个审计日志文件的最大值(以兆字节为单位)，默认大小为100M。

## 二、审计日志级别

下表显示了每个`AUDIT_LEVEL`设置，记录的API事务具体内容。

| `AUDIT_LEVEL`设置 | Request Header | Request Body | Response Header | Response Body |
| ----------------- | -------------- | ------------ | --------------- | --------------- |
| `0`               |                |              |                 |                 |
| `1`               | ✓              |              |                 |                 |
| `2`               | ✓              | ✓            |                 |                 |
| `3`               | ✓              | ✓            | ✓               | ✓               |

## 三、启用API审计日志

要启用API审计日志，请停止运行的Rancher容器，然后使用以下命令重新启动它。此命令包含打开API审计的参数，有关与API审计相关的每个`AUDIT_LEVEL`使用的详细信息，请参阅[API审计日志设置选项](#一-api审计日志设置选项)。

### 单节点安装启用

```bash
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v <主机路径>:/var/lib/rancher/ \
  -v /root/var/log/auditlog:/var/log/auditlog \
  -e AUDIT_LEVEL=3 \
  -e AUDIT_LOG_PATH=/var/log/auditlog/rancher-api-audit.log \
  -e AUDIT_LOG_MAXAGE=20 \
  -e AUDIT_LOG_MAXBACKUP=20 \
  -e AUDIT_LOG_MAXSIZE=100 \
  rancher/rancher:stable (或者rancher/rancher:latest)
```

### HA安装启用

1. RKE HA 安装 (仅支持Rancher v2.0.8之前的版本)

    在RKE 配置文件中，给Rancher容器添加以下参数:
    - 添加API审计功能参数到Rancher容器的`args`中；
    - 在容器的`volumemount`参数中声明一个`mountPath`；
    - 在`volumes`配置中声明一个`path`;

        示例配置:

        ```yaml
        ...
        containers:
            - image: rancher/rancher:stable (或者rancher/rancher:latest)
              imagePullPolicy: Always
              name: cattle-server
              args: ["--audit-log-path", "/var/log/auditlog/rancher-api-audit.log", "--audit-log-maxbackup", "5",     "--audit-log-maxsize", "50", "--audit-level", "2"]
              ports:
              - containerPort: 80
                protocol: TCP
              - containerPort: 443
                protocol: TCP
              volumeMounts:
              - mountPath: /etc/rancher/ssl
                name: cattle-keys-volume
                readOnly: true
              - mountPath: /var/log/auditlog
                name: audit-log-dir
            volumes:
            - name: cattle-keys-volume
              secret:
                defaultMode: 420
                secretName: cattle-keys-server
            - name: audit-log-dir
              hostPath:
                path: /var/log/rancher/auditlog
                type: Directory
        ```

2. Chart HA安装(适用于Rancherv2.1.0及以后版本)

    在使用Helm chart安装Rancher时启用API审计功能，会在Rancher pod中创建一个`rancher-audit-log` sidecar容器。 此容器将API审计日志发送到标准输出，可以通过查看容器日志的方式查看API审计日志`rancher-audit-log`容器位于`rancher` pod所在的`cattle-system` 命名空间中。

    启用日志审计:

    在通过`Chart`安装Rancher时，添加参数`--set auditLog.level=1`。\
    参数使用可参考[Chart设置参数]({{< baseurl >}}/rancher/v2.x/cn/installation/ha-install/helm-rancher/chart-options/)。

## 四、查看API审计日志

### Rancher单节点安装

单节点安装Rancher server时，与主机系统共享`AUDIT_LOG_PATH`目录(默认:`/var/log/auditlog`)。日志可以通过标准CLI工具进行查看，也可以转发到日志收集工具，如Fluentd，Filebeat，Logstash等。

### Rancher HA安装

在使用Helm chart安装Rancher时启用API审计功能，会在Rancher pod中创建一个`rancher-audit-log` sidecar容器。 此容器将API审计日志发送到标准输出，可以通过查看容器日志的方式查看API审计日志。

`rancher-audit-log` 容器位于`rancher` pod所在的`cattle-system` 命名空间中。

#### 通过CLI查看

```bash
kubectl --kubeconfig=kube_configxxx.yml -n cattle-system logs -f rancher-84d886bdbb-s4s69 rancher-audit-log
```

#### 通过Rancher Web GUI查看

1. 从下拉菜单中, 切换到 **Cluster: local > System**项目下

    ![Local Cluster: System Project]({{< baseurl >}}/img/rancher/audit_logs_gui/context_local_system.png)

1. 从**工作负载**菜单中，找到`cattle-system`命名空间，并找到`rancher`工作负载。

    ![Rancher Workload]({{< baseurl >}}/img/rancher/audit_logs_gui/rancher_workload.png)

1. 通过 **Ellipsis (...) > View Logs**查看`rancher` pods日志

    ![View Logs]({{< baseurl >}}/img/rancher/audit_logs_gui/view_logs.png)

1. 从**Logs**下拉菜单中，选择`rancher-audit-log`

    ![Select Audit Log]({{< baseurl >}}/img/rancher/audit_logs_gui/rancher_audit_log_container.png)

#### 收集API审计日志

可以为集群启用Rancher的内置日志收集功能，将审计和其他服务日志发送到受支持的收集服务。

## 审计日志样本

启用审核后，Rancher以JSON的形式记录每个API请求或响应。以下每个代码示例都提供了如何标识每个API事务的示例。

{{% accordion id="option-1" label="元数据级" %}}

如果设置AUDIT_LEVEL为1，Rancher会记录每个API请求的元数据标头，但不会记录正文。标题提供有关API事务的基本信息，例如事务的ID，发起事务的人员，事件发生的时间等。

```json
{
    "auditID": "30022177-9e2e-43d1-b0d0-06ef9d3db183",
    "requestURI": "/v3/schemas",
    "sourceIPs": [
        "::1"
    ],
    "user": {
        "name": "user-f4tt2",
        "group": [
            "system:authenticated"
        ]
    },
    "verb": "GET",
    "stage": "RequestReceived",
    "stageTimestamp": "2018-07-20 10:22:43 +0800"
}
```

{{% /accordion %}}
{{% accordion id="option-2" label="元数据和请求正文级别" %}}

如果设置AUDIT_LEVEL为2，Rancher会记录每个API请求的元数据标题和正文。下面的代码示例描述了一个API请求，包含其元数据头和正文。

```json
{
    "auditID": "ef1d249e-bfac-4fd0-a61f-cbdcad53b9bb",
    "requestURI": "/v3/project/c-bcz5t:p-fdr4s/workloads/deployment:default:nginx",
    "sourceIPs": [
        "::1"
    ],
    "user": {
        "name": "user-f4tt2",
        "group": [
            "system:authenticated"
        ]
    },
    "verb": "PUT",
    "stage": "RequestReceived",
    "stageTimestamp": "2018-07-20 10:28:08 +0800",
    "requestBody": {
        "hostIPC": false,
        "hostNetwork": false,
        "hostPID": false,
        "paused": false,
        "annotations": {},
        "baseType": "workload",
        "containers": [
            {
                "allowPrivilegeEscalation": false,
                "image": "nginx",
                "imagePullPolicy": "Always",
                "initContainer": false,
                "name": "nginx",
                "ports": [
                    {
                        "containerPort": 80,
                        "dnsName": "nginx-nodeport",
                        "kind": "NodePort",
                        "name": "80tcp01",
                        "protocol": "TCP",
                        "sourcePort": 0,
                        "type": "/v3/project/schemas/containerPort"
                    }
                ],
                "privileged": false,
                "readOnly": false,
                "resources": {
                    "type": "/v3/project/schemas/resourceRequirements",
                    "requests": {},
                    "limits": {}
                },
                "restartCount": 0,
                "runAsNonRoot": false,
                "stdin": true,
                "stdinOnce": false,
                "terminationMessagePath": "/dev/termination-log",
                "terminationMessagePolicy": "File",
                "tty": true,
                "type": "/v3/project/schemas/container",
                "environmentFrom": [],
                "capAdd": [],
                "capDrop": [],
                "livenessProbe": null,
                "volumeMounts": []
            }
        ],
        "created": "2018-07-18T07:34:16Z",
        "createdTS": 1531899256000,
        "creatorId": null,
        "deploymentConfig": {
            "maxSurge": 1,
            "maxUnavailable": 0,
            "minReadySeconds": 0,
            "progressDeadlineSeconds": 600,
            "revisionHistoryLimit": 10,
            "strategy": "RollingUpdate"
        },
        "deploymentStatus": {
            "availableReplicas": 1,
            "conditions": [
                {
                    "lastTransitionTime": "2018-07-18T07:34:38Z",
                    "lastTransitionTimeTS": 1531899278000,
                    "lastUpdateTime": "2018-07-18T07:34:38Z",
                    "lastUpdateTimeTS": 1531899278000,
                    "message": "Deployment has minimum availability.",
                    "reason": "MinimumReplicasAvailable",
                    "status": "True",
                    "type": "Available"
                },
                {
                    "lastTransitionTime": "2018-07-18T07:34:16Z",
                    "lastTransitionTimeTS": 1531899256000,
                    "lastUpdateTime": "2018-07-18T07:34:38Z",
                    "lastUpdateTimeTS": 1531899278000,
                    "message": "ReplicaSet \"nginx-64d85666f9\" has successfully progressed.",
                    "reason": "NewReplicaSetAvailable",
                    "status": "True",
                    "type": "Progressing"
                }
            ],
            "observedGeneration": 2,
            "readyReplicas": 1,
            "replicas": 1,
            "type": "/v3/project/schemas/deploymentStatus",
            "unavailableReplicas": 0,
            "updatedReplicas": 1
        },
        "dnsPolicy": "ClusterFirst",
        "id": "deployment:default:nginx",
        "labels": {
            "workload.user.cattle.io/workloadselector": "deployment-default-nginx"
        },
        "name": "nginx",
        "namespaceId": "default",
        "projectId": "c-bcz5t:p-fdr4s",
        "publicEndpoints": [
            {
                "addresses": [
                    "10.64.3.58"
                ],
                "allNodes": true,
                "ingressId": null,
                "nodeId": null,
                "podId": null,
                "port": 30917,
                "protocol": "TCP",
                "serviceId": "default:nginx-nodeport",
                "type": "publicEndpoint"
            }
        ],
        "restartPolicy": "Always",
        "scale": 1,
        "schedulerName": "default-scheduler",
        "selector": {
            "matchLabels": {
                "workload.user.cattle.io/workloadselector": "deployment-default-nginx"
            },
            "type": "/v3/project/schemas/labelSelector"
        },
        "state": "active",
        "terminationGracePeriodSeconds": 30,
        "transitioning": "no",
        "transitioningMessage": "",
        "type": "deployment",
        "uuid": "f998037d-8a5c-11e8-a4cf-0245a7ebb0fd",
        "workloadAnnotations": {
            "deployment.kubernetes.io/revision": "1",
            "field.cattle.io/creatorId": "user-f4tt2"
        },
        "workloadLabels": {
            "workload.user.cattle.io/workloadselector": "deployment-default-nginx"
        },
        "scheduling": {
            "node": {}
        },
        "description": "my description",
        "volumes": []
    }
}
```

{{% /accordion %}}
{{% accordion id="option-3" label="元数据、请求正文和响应正文级别" %}}

如果您设置AUDIT_LEVEL为3，Rancher将记录：

- 每个API请求的元数据标头和正文。
- 每个API响应的元数据标头和正文。

{{% accordion id="option-4" label="请求" %}}

下面的代码示例描述了一个API请求，它有元数据头和正文。

```json
{
    "auditID": "a886fd9f-5d6b-4ae3-9a10-5bff8f3d68af",
    "requestURI": "/v3/project/c-bcz5t:p-fdr4s/workloads/deployment:default:nginx",
    "sourceIPs": [
        "::1"
    ],
    "user": {
        "name": "user-f4tt2",
        "group": [
            "system:authenticated"
        ]
    },
    "verb": "PUT",
    "stage": "RequestReceived",
    "stageTimestamp": "2018-07-20 10:33:06 +0800",
    "requestBody": {
        "hostIPC": false,
        "hostNetwork": false,
        "hostPID": false,
        "paused": false,
        "annotations": {},
        "baseType": "workload",
        "containers": [
            {
                "allowPrivilegeEscalation": false,
                "image": "nginx",
                "imagePullPolicy": "Always",
                "initContainer": false,
                "name": "nginx",
                "ports": [
                    {
                        "containerPort": 80,
                        "dnsName": "nginx-nodeport",
                        "kind": "NodePort",
                        "name": "80tcp01",
                        "protocol": "TCP",
                        "sourcePort": 0,
                        "type": "/v3/project/schemas/containerPort"
                    }
                ],
                "privileged": false,
                "readOnly": false,
                "resources": {
                    "type": "/v3/project/schemas/resourceRequirements",
                    "requests": {},
                    "limits": {}
                },
                "restartCount": 0,
                "runAsNonRoot": false,
                "stdin": true,
                "stdinOnce": false,
                "terminationMessagePath": "/dev/termination-log",
                "terminationMessagePolicy": "File",
                "tty": true,
                "type": "/v3/project/schemas/container",
                "environmentFrom": [],
                "capAdd": [],
                "capDrop": [],
                "livenessProbe": null,
                "volumeMounts": []
            }
        ],
        "created": "2018-07-18T07:34:16Z",
        "createdTS": 1531899256000,
        "creatorId": null,
        "deploymentConfig": {
            "maxSurge": 1,
            "maxUnavailable": 0,
            "minReadySeconds": 0,
            "progressDeadlineSeconds": 600,
            "revisionHistoryLimit": 10,
            "strategy": "RollingUpdate"
        },
        "deploymentStatus": {
            "availableReplicas": 1,
            "conditions": [
                {
                    "lastTransitionTime": "2018-07-18T07:34:38Z",
                    "lastTransitionTimeTS": 1531899278000,
                    "lastUpdateTime": "2018-07-18T07:34:38Z",
                    "lastUpdateTimeTS": 1531899278000,
                    "message": "Deployment has minimum availability.",
                    "reason": "MinimumReplicasAvailable",
                    "status": "True",
                    "type": "Available"
                },
                {
                    "lastTransitionTime": "2018-07-18T07:34:16Z",
                    "lastTransitionTimeTS": 1531899256000,
                    "lastUpdateTime": "2018-07-18T07:34:38Z",
                    "lastUpdateTimeTS": 1531899278000,
                    "message": "ReplicaSet \"nginx-64d85666f9\" has successfully progressed.",
                    "reason": "NewReplicaSetAvailable",
                    "status": "True",
                    "type": "Progressing"
                }
            ],
            "observedGeneration": 2,
            "readyReplicas": 1,
            "replicas": 1,
            "type": "/v3/project/schemas/deploymentStatus",
            "unavailableReplicas": 0,
            "updatedReplicas": 1
        },
        "dnsPolicy": "ClusterFirst",
        "id": "deployment:default:nginx",
        "labels": {
            "workload.user.cattle.io/workloadselector": "deployment-default-nginx"
        },
        "name": "nginx",
        "namespaceId": "default",
        "projectId": "c-bcz5t:p-fdr4s",
        "publicEndpoints": [
            {
                "addresses": [
                    "10.64.3.58"
                ],
                "allNodes": true,
                "ingressId": null,
                "nodeId": null,
                "podId": null,
                "port": 30917,
                "protocol": "TCP",
                "serviceId": "default:nginx-nodeport",
                "type": "publicEndpoint"
            }
        ],
        "restartPolicy": "Always",
        "scale": 1,
        "schedulerName": "default-scheduler",
        "selector": {
            "matchLabels": {
                "workload.user.cattle.io/workloadselector": "deployment-default-nginx"
            },
            "type": "/v3/project/schemas/labelSelector"
        },
        "state": "active",
        "terminationGracePeriodSeconds": 30,
        "transitioning": "no",
        "transitioningMessage": "",
        "type": "deployment",
        "uuid": "f998037d-8a5c-11e8-a4cf-0245a7ebb0fd",
        "workloadAnnotations": {
            "deployment.kubernetes.io/revision": "1",
            "field.cattle.io/creatorId": "user-f4tt2"
        },
        "workloadLabels": {
            "workload.user.cattle.io/workloadselector": "deployment-default-nginx"
        },
        "scheduling": {
            "node": {}
        },
        "description": "my decript",
        "volumes": []
    }
}
```

{{% /accordion %}}
{{% accordion id="option-5" label="响应" %}}

下面的代码示例描述了一个API响应，其中包含它的元数据头和正文。

```json
{
    "auditID": "a886fd9f-5d6b-4ae3-9a10-5bff8f3d68af",
    "responseStatus": "200",
    "stage": "ResponseComplete",
    "stageTimestamp": "2018-07-20 10:33:06 +0800",
    "responseBody": {
        "actionLinks": {
            "pause": "https://localhost:8443/v3/project/c-bcz5t:p-fdr4s/workloads/deployment:default:nginx?action=pause",
            "resume": "https://localhost:8443/v3/project/c-bcz5t:p-fdr4s/workloads/deployment:default:nginx?action=resume",
            "rollback": "https://localhost:8443/v3/project/c-bcz5t:p-fdr4s/workloads/deployment:default:nginx?action=rollback"
        },
        "annotations": {},
        "baseType": "workload",
        "containers": [
            {
                "allowPrivilegeEscalation": false,
                "image": "nginx",
                "imagePullPolicy": "Always",
                "initContainer": false,
                "name": "nginx",
                "ports": [
                    {
                        "containerPort": 80,
                        "dnsName": "nginx-nodeport",
                        "kind": "NodePort",
                        "name": "80tcp01",
                        "protocol": "TCP",
                        "sourcePort": 0,
                        "type": "/v3/project/schemas/containerPort"
                    }
                ],
                "privileged": false,
                "readOnly": false,
                "resources": {
                    "type": "/v3/project/schemas/resourceRequirements"
                },
                "restartCount": 0,
                "runAsNonRoot": false,
                "stdin": true,
                "stdinOnce": false,
                "terminationMessagePath": "/dev/termination-log",
                "terminationMessagePolicy": "File",
                "tty": true,
                "type": "/v3/project/schemas/container"
            }
        ],
        "created": "2018-07-18T07:34:16Z",
        "createdTS": 1531899256000,
        "creatorId": null,
        "deploymentConfig": {
            "maxSurge": 1,
            "maxUnavailable": 0,
            "minReadySeconds": 0,
            "progressDeadlineSeconds": 600,
            "revisionHistoryLimit": 10,
            "strategy": "RollingUpdate"
        },
        "deploymentStatus": {
            "availableReplicas": 1,
            "conditions": [
                {
                    "lastTransitionTime": "2018-07-18T07:34:38Z",
                    "lastTransitionTimeTS": 1531899278000,
                    "lastUpdateTime": "2018-07-18T07:34:38Z",
                    "lastUpdateTimeTS": 1531899278000,
                    "message": "Deployment has minimum availability.",
                    "reason": "MinimumReplicasAvailable",
                    "status": "True",
                    "type": "Available"
                },
                {
                    "lastTransitionTime": "2018-07-18T07:34:16Z",
                    "lastTransitionTimeTS": 1531899256000,
                    "lastUpdateTime": "2018-07-18T07:34:38Z",
                    "lastUpdateTimeTS": 1531899278000,
                    "message": "ReplicaSet \"nginx-64d85666f9\" has successfully progressed.",
                    "reason": "NewReplicaSetAvailable",
                    "status": "True",
                    "type": "Progressing"
                }
            ],
            "observedGeneration": 2,
            "readyReplicas": 1,
            "replicas": 1,
            "type": "/v3/project/schemas/deploymentStatus",
            "unavailableReplicas": 0,
            "updatedReplicas": 1
        },
        "dnsPolicy": "ClusterFirst",
        "hostIPC": false,
        "hostNetwork": false,
        "hostPID": false,
        "id": "deployment:default:nginx",
        "labels": {
            "workload.user.cattle.io/workloadselector": "deployment-default-nginx"
        },
        "links": {
            "remove": "https://localhost:8443/v3/project/c-bcz5t:p-fdr4s/workloads/deployment:default:nginx",
            "revisions": "https://localhost:8443/v3/project/c-bcz5t:p-fdr4s/workloads/deployment:default:nginx/revisions",
            "self": "https://localhost:8443/v3/project/c-bcz5t:p-fdr4s/workloads/deployment:default:nginx",
            "update": "https://localhost:8443/v3/project/c-bcz5t:p-fdr4s/workloads/deployment:default:nginx",
            "yaml": "https://localhost:8443/v3/project/c-bcz5t:p-fdr4s/workloads/deployment:default:nginx/yaml"
        },
        "name": "nginx",
        "namespaceId": "default",
        "paused": false,
        "projectId": "c-bcz5t:p-fdr4s",
        "publicEndpoints": [
            {
                "addresses": [
                    "10.64.3.58"
                ],
                "allNodes": true,
                "ingressId": null,
                "nodeId": null,
                "podId": null,
                "port": 30917,
                "protocol": "TCP",
                "serviceId": "default:nginx-nodeport"
            }
        ],
        "restartPolicy": "Always",
        "scale": 1,
        "schedulerName": "default-scheduler",
        "selector": {
            "matchLabels": {
                "workload.user.cattle.io/workloadselector": "deployment-default-nginx"
            },
            "type": "/v3/project/schemas/labelSelector"
        },
        "state": "active",
        "terminationGracePeriodSeconds": 30,
        "transitioning": "no",
        "transitioningMessage": "",
        "type": "deployment",
        "uuid": "f998037d-8a5c-11e8-a4cf-0245a7ebb0fd",
        "workloadAnnotations": {
            "deployment.kubernetes.io/revision": "1",
            "field.cattle.io/creatorId": "user-f4tt2"
        },
        "workloadLabels": {
            "workload.user.cattle.io/workloadselector": "deployment-default-nginx"
        }
    }
}
```

{{% /accordion %}}
{{% /accordion %}}
