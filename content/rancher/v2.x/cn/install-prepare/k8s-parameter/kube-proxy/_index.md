---
title: 3 - kube-proxy
weight: 3
---

### Synopsis

The Kubernetes network proxy runs on each node. This reflects services as defined in the Kubernetes API on each node and can do simple TCP, UDP, and SCTP stream forwarding or round robin TCP, UDP, and SCTP forwarding across a set of backends. Service cluster IPs and ports are currently found through Docker-links-compatible environment variables specifying ports opened by the service proxy. There is an optional addon that provides cluster DNS for these cluster IPs. The user must create a service with the apiserver API to configure the proxy.

Kubernetes网络代理在每个节点上运行。这反映了Kubernetes API在每个节点上定义的服务，可以跨一组后端执行简单的TCP、UDP和SCTP流转发或轮询TCP、UDP和SCTP转发。服务集群ip和端口目前是通过指定由服务代理打开的端口的Docker-links-compatible environment变量找到的。有一个可选的插件为这些集群ip提供集群DNS。用户必须使用apiserver API创建一个服务来配置代理。

```bash
kube-proxy [flags]
```

### Options

| --azure-container-registry-config string                     | 包含Azure容器镜像仓库配置信息的文件的路径。                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| --bind-address 0.0.0.0     Default: 0.0.0.0                  | 代理服务器要服务的IP地址(对于所有IPv4接口设置为0.0.0.0，对于所有IPv6接口设置为'::') |
| --cleanup                                                    | 如果true，清除iptables和ipvs规则并退出。                     |
| --cleanup-ipvs     Default: true                             | 如果为true，在运行前清除kube-proxy ipvs规则。默认 **true**   |
| --cluster-cidr string                                        | 集群中pod的CIDR范围。配置好后，从该范围之外发送到服务cluster IP的流量将被伪装，而从pods发送到外部LoadBalancer IP的流量将被定向到相应的cluster IP |
| --config string                                              | 配置文件的路径。                                             |
| --config-sync-period duration     Default: 15m0s             | 从apiserver中刷新配置的频率。必须大于0。                     |
| --conntrack-max-per-core int32     Default: 32768            | 要跟踪每个CPU内核的NAT连接的最大数量(0保留限制为原样，忽略conntrack-min)。 |
| --conntrack-min int32     Default: 131072                    | 分配的conntrack条目的最小数量，而不考虑conntrack-max-per-core(将conntrack-max-per-core设置为0以保持限制不变)。 |
| --conntrack-tcp-timeout-close-wait duration     Default: 1h0m0s | 处于CLOSE_WAIT状态的TCP连接的NAT超时时间                     |
| --conntrack-tcp-timeout-established duration     Default: 24h0m0s | 已建立TCP连接的空闲超时(0保留原样)                           |
| --feature-gates mapStringBool                                | A set of key=value pairs that describe feature gates for alpha/experimental features. Options are: APIListChunking=true\|false (BETA - default=true) APIResponseCompression=true\|false (ALPHA - default=false) AllAlpha=true\|false (ALPHA - default=false) AppArmor=true\|false (BETA - default=true) AttachVolumeLimit=true\|false (BETA - default=true) BalanceAttachedNodeVolumes=true\|false (ALPHA - default=false) BlockVolume=true\|false (BETA - default=true) BoundServiceAccountTokenVolume=true\|false (ALPHA - default=false) CPUManager=true\|false (BETA - default=true) CRIContainerLogRotation=true\|false (BETA - default=true) CSIBlockVolume=true\|false (BETA - default=true) CSIDriverRegistry=true\|false (BETA - default=true) CSIInlineVolume=true\|false (ALPHA - default=false) CSIMigration=true\|false (ALPHA - default=false) CSIMigrationAWS=true\|false (ALPHA - default=false) CSIMigrationGCE=true\|false (ALPHA - default=false) CSIMigrationOpenStack=true\|false (ALPHA - default=false) CSINodeInfo=true\|false (BETA - default=true) CustomCPUCFSQuotaPeriod=true\|false (ALPHA - default=false) CustomResourcePublishOpenAPI=true\|false (ALPHA - default=false) CustomResourceSubresources=true\|false (BETA - default=true) CustomResourceValidation=true\|false (BETA - default=true) CustomResourceWebhookConversion=true\|false (ALPHA - default=false) DebugContainers=true\|false (ALPHA - default=false) DevicePlugins=true\|false (BETA - default=true) DryRun=true\|false (BETA - default=true) DynamicAuditing=true\|false (ALPHA - default=false) DynamicKubeletConfig=true\|false (BETA - default=true) ExpandCSIVolumes=true\|false (ALPHA - default=false) ExpandInUsePersistentVolumes=true\|false (ALPHA - default=false) ExpandPersistentVolumes=true\|false (BETA - default=true) ExperimentalCriticalPodAnnotation=true\|false (ALPHA - default=false) ExperimentalHostUserNamespaceDefaulting=true\|false (BETA - default=false) HyperVContainer=true\|false (ALPHA - default=false) KubeletPodResources=true\|false (ALPHA - default=false) LocalStorageCapacityIsolation=true\|false (BETA - default=true) MountContainers=true\|false (ALPHA - default=false) NodeLease=true\|false (BETA - default=true) PodShareProcessNamespace=true\|false (BETA - default=true) ProcMountType=true\|false (ALPHA - default=false) QOSReserved=true\|false (ALPHA - default=false) ResourceLimitsPriorityFunction=true\|false (ALPHA - default=false) ResourceQuotaScopeSelectors=true\|false (BETA - default=true) RotateKubeletClientCertificate=true\|false (BETA - default=true) RotateKubeletServerCertificate=true\|false (BETA - default=true) RunAsGroup=true\|false (BETA - default=true) RuntimeClass=true\|false (BETA - default=true) SCTPSupport=true\|false (ALPHA - default=false) ScheduleDaemonSetPods=true\|false (BETA - default=true) ServerSideApply=true\|false (ALPHA - default=false) ServiceNodeExclusion=true\|false (ALPHA - default=false) StorageVersionHash=true\|false (ALPHA - default=false) StreamingProxyRedirects=true\|false (BETA - default=true) SupportNodePidsLimit=true\|false (ALPHA - default=false) SupportPodPidsLimit=true\|false (BETA - default=true) Sysctls=true\|false (BETA - default=true) TTLAfterFinished=true\|false (ALPHA - default=false) TaintBasedEvictions=true\|false (BETA - default=true) TaintNodesByCondition=true\|false (BETA - default=true) TokenRequest=true\|false (BETA - default=true) TokenRequestProjection=true\|false (BETA - default=true) ValidateProxyRedirects=true\|false (BETA - default=true) VolumeSnapshotDataSource=true\|false (ALPHA - default=false) VolumeSubpathEnvExpansion=true\|false (ALPHA - default=false) WinDSR=true\|false (ALPHA - default=false) WinOverlay=true\|false (ALPHA - default=false) WindowsGMSA=true\|false (ALPHA - default=false) |
| --healthz-bind-address 0.0.0.0     Default: 0.0.0.0:10256    | 健康检查服务器的IP地址(对于所有IPv4接口设置为0.0.0.0，对于所有IPv6接口设置为'::') |
| --healthz-port int32     Default: 10256                      | 绑定健康检查服务器的端口。使用0禁用。                        |
| -h, --help                                                   | help for kube-proxy                                          |
| --hostname-override string                                   | 如果非空，将使用此字符串作为标识符，而不是实际的主机名。     |
| --iptables-masquerade-bit int32     Default: 14              | 如果使用纯iptables代理，则使用fwmark空间的位来标记需要SNAT的包。必须在[0,31]范围内。 |
| --iptables-min-sync-period duration                          | iptables规则在端点和服务发生更改时刷新的频率的最小间隔 (e.g. '5s', '1m', '2h22m'). |
| --iptables-sync-period duration     Default: 30s             | iptables规则刷新频率的最大间隔(e.g. '5s', '1m', '2h22m'). 必须大于0。 |
| --ipvs-exclude-cidrs stringSlice                             | 一个逗号分隔的CIDR列表，ipvs代理程序在清理ipvs规则时不会清楚 |
| --ipvs-min-sync-period duration                              | 当端点和服务发生变化时，ipvs规则能够被刷新的最小间隔 (e.g. '5s', '1m', '2h22m'). |
| --ipvs-scheduler string                                      | 代理模式为ipvs时的ipvs调度程序类型                           |
| --ipvs-sync-period duration     Default: 30s                 | ipvs规则刷新频率的最大频率 (e.g. '5s', '1m', '2h22m'). 必须大于0。 |
| --kube-api-burst int32     Default: 10                       | 与kubernetes apiserver通信并发数                             |
| --kube-api-content-type string     Default: "application/vnd.kubernetes.protobuf" | 发送到apiserver的请求的内容类型。                            |
| --kube-api-qps float32     Default: 5                        | 与kubernetes apiserver通信时使用QPS                          |
| --kubeconfig string                                          | 带有授权信息的kubeconfig文件的路径(主位置由主标志设置)。     |
| --log-flush-frequency duration     Default: 5s               | 日志刷新之间的最大秒数                                       |
| --masquerade-all                                             | If using the pure iptables proxy, SNAT all traffic sent via Service cluster IPs (this not commonly needed) |
| --master string                                              | The address of the Kubernetes API server (overrides any value in kubeconfig) |
| --metrics-bind-address 0.0.0.0     Default: 127.0.0.1:10249  | The IP address for the metrics server to serve on (set to 0.0.0.0 for all IPv4 interfaces and `::` for all IPv6 interfaces) |
| --metrics-port int32     Default: 10249                      | The port to bind the metrics server. Use 0 to disable.       |
| --nodeport-addresses stringSlice                             | A string slice of values which specify the addresses to use for NodePorts. Values may be valid IP blocks (e.g. 1.2.3.0/24, 1.2.3.4/32). The default empty string slice ([]) means to use all local addresses. |
| --oom-score-adj int32     Default: -999                      | The oom-score-adj value for kube-proxy process. Values must be within the range [-1000, 1000] |
| --profiling                                                  | 如果为true，则启用通过`/debug/pprof`处理程序上的web接口进行分析。 |
| --proxy-mode ProxyMode                                       | 使用哪一种代理模式:`userspace`或`iptables`或`ipvs`。如果为空，则使用可用的最佳模式。如果选择了iptables模式，但是系统的内核或iptables版本不满足要求，那么会自动切换到`userspace`模式。 |
| --proxy-port-range port-range                                | Range of host ports (beginPort-endPort, single port or beginPort+offset, inclusive) that may be consumed in order to proxy service traffic. If (unspecified, 0, or 0-0) then ports will be randomly chosen. |
| --udp-timeout duration     Default: 250ms                    | How long an idle UDP connection will be kept open (e.g. '250ms', '2s'). Must be greater than 0. Only applicable for proxy-mode=userspace |
| --version version[=true]                                     | Print version information and quit                           |
| --write-config-to string                                     | If set, write the default configuration values to this file and exit. |