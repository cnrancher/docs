---
title: 6 - API审计日志
weight: 6
---

Rancher附带API审计功能，用以记录每个用户发起的系统事件信息。你可以知道发生了什么、何时发生、谁发起的、以及事件对集群的影响。API审计也记录Rancher API的所有请求和响应，包括使用Rancher UI和通过编程使用Rancher API的所有其他用途。

你可以在Rancher安装或升级期间启用API审计。

## API审计使用方法

以下定义了有关审核日志记录的内容以及包含哪些数据的规则：

参数 | 描述
---------|----------
 `AUDIT_LEVEL` | `0` - 禁用审核日志(默认设置)<br/>`1` - 记录事件元数据<br/>`2` - 记录事件元数据和请求内容. <br/>`3` - 记录事件元数据、请求内容和响应内容。请求/响应对的每个日志事务使用相同的`auditID`值。<br/>有关显示每个等级设置记录的具体内容，请参阅[审计级别日志](#审计日志级别)记录
 `AUDIT_LOG_PATH` | Rancher Server API的日志路径，默认路径是：`/var/log/auditlog/rancher-api-audit.log`，你可以将日志目录挂载到主机。<br/>用法示例: `AUDIT_LOG_PATH=/my/custom/path/`<br/> 
 `AUDIT_LOG_MAXAGE` | 定义了保留旧审核日志文件的最大天数，默认为10天。
 `AUDIT_LOG_MAXBACKUP` | 定义要保留的最大审核日志文件个数，默认值为10。
 `AUDIT_LOG_MAXSIZE` | 定义单个审计日志文件的最大值(以兆字节为单位)，默认大小为100M。

## 审计日志级别

下表显示了每个`AUDIT_LEVEL`设置，记录的API事务具体内容。

| `AUDIT_LEVEL`设置 | Request Header | Request Body | Response Header | Response Header |
| ----------------- | -------------- | ------------ | --------------- | --------------- |
| `0`               |                |              |                 |                 |
| `1`               | ✓              |              |                 |                 |
| `2`               | ✓              | ✓            |                 |                 |
| `3`               | ✓              | ✓            | ✓               | ✓               |

## 启用API审核

要启用API审核，请停止运行的Rancher容器，然后使用以下命令重新启动它。此命令包含打开API审核的参数，有关与API审核相关的每个`AUDIT_LEVEL`使用的详细信息，请参阅[API审计使用方法](#API审计使用方法)。

```bash
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /root/var/log/auditlog:/var/log/auditlog \
  -e AUDIT_LEVEL=1 \
  -e AUDIT_LOG_PATH=/var/log/auditlog/rancher-api-audit.log \
  -e AUDIT_LOG_MAXAGE=20 \
  -e AUDIT_LOG_MAXBACKUP=20 \
  -e AUDIT_LOG_MAXSIZE=100 \
  rancher/rancher:latest
```

## 查看API审核日志

默认情况下，你可以使用自己熟悉的文本编辑器在Rancher server节点上查看审核日志。例如：

```bash
less /root/var/log/auditlog/rancher-api-audit.log
```

## 审核日志样本

启用审核后，Rancher以JSON的形式记录每个API请求或响应。以下每个代码示例都提供了如何标识每个API事务的示例。

### 元数据级别

如果设置`AUDIT_LEVEL`为`1`，Rancher会记录每个API请求的元数据标头，但不会记录正文内容。标题提供有关API事务的基本信息，例如事务的ID，发起事务的人员，事件发生的时间等。

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

### 元数据和请求内容级别

如果设置`AUDIT_LEVEL`为`2`，Rancher会记录每个API请求的元数据标题和正文内容。

下面的代码示例描述了一个API请求，包含其元数据头和正文内容。

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

### 元数据，请求内容和响应内容级别

如果你设置`AUDIT_LEVEL`为`3`，Rancher记录：

- 每个API请求的元数据标头和正文内容。
- 每个API响应的元数据标头和正文内容。

#### 请求

下面的代码示例描述了一个API请求，包含其元数据头和正文内容。 

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

#### 响应

下面的代码示例描述了API响应，包括其元数据头和正文内容。 

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