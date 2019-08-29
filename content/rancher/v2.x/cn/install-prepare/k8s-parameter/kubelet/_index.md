---
title: 5 - kubelet
weight: 5
---

*当前版本v1.14*

## 简介

kubelet主要以`节点代理`的形式在每个节点上运行。 kubelet根据PodSpec工作，PodSpec是描述pod的YAML或JSON对象。kubelet接受一组通过各种机制(主要是通过apiserver)提供的podspec，并确保这些podspec中描述的容器正常运行。kubelet不管理不是由Kubernetes创建的容器。

除了来自apiserver的PodSpec之外，还有三种方法可以将容器清单提供给Kubelet。

**文件:** 在命令行上作为参数传递的路径。此路径下的文件将定期监视更新。监视周期默认为20s，可以通过一个标志进行配置。

**HTTP endpoint:** 在命令行上作为参数传递的HTTP端点。此端点每20秒检查一次(也可以使用标志进行配置)。

**HTTP server:** kubelet还可以侦听HTTP并响应一个简单的API(目前未达到规范)来提交一个新的清单。

### Pod生命周期事件生成器(PLEG)

Pod生命周期事件生成器是kubelet的一个功能，它为所有容器和Pod创建一个状态列表，然后在一个名为Relisting的过程中将其与容器和Pod的先前状态进行比较。这允许PLEG知道哪些PODs和容器需要同步。在1.2之前的版本中，这是通过轮询来完成的，并且是CPU密集型的。通过更改此方法，可以显著降低资源利用率，从而提高容器密度.

```bash
kubelet [flags]
```

## 参数选项

| --address 0.0.0.0                                            | Kubelet要使用的IP地址(对于所有IPv4接口设置为0.0.0.0，对于所有IPv6接口设置为'::')(默认值0.0.0.0) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| --allow-privileged                                           | 如果为 true ，则允许容器请求特权模式。                       |
| --alsologtostderr                                            | log to standard error as well as files                       |
| --anonymous-auth                                             | 启用对Kubelet服务器的匿名请求。未被其他身份验证方法拒绝的请求被视为匿名请求。<br />匿名请求的用户名是system: Anonymous，组名是system:unauthenticated。(默认 true) |
| --application-metrics-count-limit int                        | 要存储的应用程序度量的最大数量(每个容器)(默认为100)          |
| --authentication-token-webhook                               | 使用TokenReview API来确定承载令牌的身份验证。                |
| --authentication-token-webhook-cache-ttl duration            | 缓存来自webhook令牌验证器的响应的持续时间。(默认 2m0s )      |
| --authorization-mode string                                  | Kubelet服务器的授权模式。有效的选项**AlwaysAllow**或**Webhook**。<br />Webhook模式使用SubjectAccessReview API来确定授权。(默认“AlwaysAllow”) |
| --authorization-webhook-cache-authorized-ttl duration        | 缓存来自webhook授权器的“已授权”响应的持续时间。(默认 5m0s)   |
| --authorization-webhook-cache-unauthorized-ttl duration      | 缓存来自webhook授权器的“未授权”响应的持续时间。(default 30s) |
| --azure-container-registry-config string                     | 包含Azure容器镜像仓库配置信息的文件的路径。                  |
| --boot-id-file string                                        | 逗号分隔的文件列表，用于检查引导id。使用现有的第一个。(default "/proc/sys/kernel/random/boot_id") |
| --bootstrap-checkpoint-path string                           | 指向存储检查点的目录路径                                     |
| --bootstrap-kubeconfig string                                | Path to a kubeconfig file that will be used to get client certificate for kubelet. If the file specified by --kubeconfig does not exist, the bootstrap kubeconfig is used to request a client certificate from the API server. On success, a kubeconfig file referencing the generated client certificate and key is written to the path specified by --kubeconfig. The client certificate and key file will be stored in the directory pointed by --cert-dir. |
| --cert-dir string                                            | The directory where the TLS certs are located. If --tls-cert-file and --tls-private-key-file are provided, this flag will be ignored. (default "/var/lib/kubelet/pki") |
| --cgroup-driver string                                       | Driver that the kubelet uses to manipulate cgroups on the host. |
| --cgroup-root string                                         | Optional root cgroup to use for pods. This is handled by the container runtime on a best effort basis. Default: '', which means use the container runtime default. |
| --cgroups-per-qos                                            | Enable creation of QoS cgroup hierarchy, if true top level QoS and pod cgroups are created. (default true) |
| --chaos-chance float                                         | If > 0.0, introduce random client errors and latency. Intended for testing. |
| --client-ca-file string                                      | If set, any request presenting a client certificate signed by one of the authorities in the client-ca-file is authenticated with an identity corresponding to the CommonName of the client certificate. |
| --cloud-config string                                        | The path to the cloud provider configuration file.           |
| --cloud-provider string                                      | The provider for cloud services. Specify empty string for running with no cloud provider. |
| --cloud-provider-gce-lb-src-cidrs cidrs                      | CIDRs opened in GCE firewall for LB traffic proxy & health checks (default 130.211.0.0/22,35.191.0.0/16,209.85.152.0/22,209.85.204.0/22) |
| --cluster-dns stringSlice                                    | 逗号分隔的DNS服务器IP地址列表。                              |
| --cluster-domain string                                      | 群集域。                                                     |
| --cni-bin-dir string                                         | 用于搜索CNI插件二进制文件的目录的完整路径。Default: **/opt/cni/bin** |
| --cni-conf-dir string                                        | The full path of the directory in which to search for CNI config files. Default: /etc/cni/net.d |
| --container-hints string                                     | location of the container hints file (default "/etc/cadvisor/container_hints.json") |
| --container-runtime string                                   | The container runtime to use. Possible values: 'docker', 'remote', 'rkt(deprecated)'. (default "docker") |
| --container-runtime-endpoint string                          | [实验]The endpoint of remote runtime service. Currently unix socket is supported on Linux, and tcp is supported on windows. |
| --containerd string                                          | containerd endpoint (default "unix:///var/run/containerd.sock") |
| --containerized                                              | Experimental support for running kubelet in a container.     |
| --contention-profiling                                       | Enable lock contention profiling, if profiling is enabled    |
| --cpu-cfs-quota                                              | Enable CPU CFS quota enforcement for containers that specify CPU limits (default true) |
| --cpu-manager-policy string                                  | CPU Manager policy to use. Possible values: 'none', 'static'. (default "none") |
| --cpu-manager-reconcile-period NodeStatusUpdateFrequency     | CPU Manager reconciliation period. Examples: '10s', or '1m'. If not supplied, defaults to NodeStatusUpdateFrequency (default 10s) |
| --docker string                                              | docker endpoint (default "unix:///var/run/docker.sock")      |
| --docker-disable-shared-pid                                  | The Container Runtime Interface (CRI) defaults to using a shared PID namespace for containers in a pod when running with Docker 1.13.1 or higher. Setting this flag reverts to the previous behavior of isolated PID namespaces. This ability will be removed in a future Kubernetes release. (default true) |
| --docker-endpoint string                                     | Use this for the docker endpoint to communicate with (default "unix:///var/run/docker.sock") |
| --docker-env-metadata-whitelist string                       | a comma-separated list of environment variable keys that needs to be collected for docker containers |
| --docker-only                                                | Only report docker containers in addition to root stats      |
| --docker-root string                                         | DEPRECATED: docker root is read from docker info (this is a fallback, default: /var/lib/docker) (default "/var/lib/docker") |
| --docker-tls                                                 | use TLS to connect to docker                                 |
| --docker-tls-ca string                                       | path to trusted CA (default "ca.pem")                        |
| --docker-tls-cert string                                     | path to client certificate (default "cert.pem")              |
| --docker-tls-key string                                      | path to private key (default "key.pem")                      |
| --dynamic-config-dir string                                  | The Kubelet will use this directory for checkpointing downloaded configurations and tracking configuration health. The Kubelet will create this directory if it does not already exist. The path may be absolute or relative; relative paths start at the Kubelet's current working directory. Providing this flag enables dynamic Kubelet configuration. Presently, you must also enable the DynamicKubeletConfig feature gate to pass this flag. |
| --enable-controller-attach-detach                            | Enables the Attach/Detach controller to manage attachment/detachment of volumes scheduled to this node, and disables kubelet from executing any attach/detach operations (default true) |
| --enable-debugging-handlers                                  | Enables server endpoints for log collection and local running of containers and commands (default true) |
| --enable-load-reader                                         | Whether to enable cpu load reader                            |
| --enable-server                                              | Enable the Kubelet's server (default true)                   |
| --enforce-node-allocatable stringSlice                       | A comma separated list of levels of node allocatable enforcement to be enforced by kubelet. Acceptable options are 'pods', 'system-reserved' & 'kube-reserved'. If the latter two options are specified, '--system-reserved-cgroup' & '--kube-reserved-cgroup' must also be set respectively. See /docs/tasks/administer-cluster/reserve-compute-resources/ for more details. (default [pods]) |
| --event-burst int32                                          | Maximum size of a bursty event records, temporarily allows event records to burst to this number, while still not exceeding event-qps. Only used if --event-qps > 0 (default 10) |
| --event-qps int32                                            | If > 0, limit event creations per second to this value. If 0, unlimited. (default 5) |
| --event-storage-age-limit string                             | Max length of time for which to store events (per type). Value is a comma separated list of key values, where the keys are event types (e.g.: creation, oom) or "default" and the value is a duration. Default is applied to all non-specified event types (default "default=0") |
| --event-storage-event-limit string                           | Max number of events to store (per type). Value is a comma separated list of key values, where the keys are event types (e.g.: creation, oom) or "default" and the value is an integer. Default is applied to all non-specified event types (default "default=0") |
| --eviction-hard mapStringString                              | 一组驱逐阈值(例如memory.available<1Gi)，如果满足这些阈值，就会触发pod驱逐。.<br /> (default imagefs.available<15%,memory.available<100Mi,nodefs.available<10%,nodefs.inodesFree<5%) |
| --eviction-max-pod-grace-period int32                        | 最大允许宽限期(秒)使用时，终止pod，以响应触发的软驱逐阈值。  |
| --eviction-minimum-reclaim mapStringString                   | 一组最小回收(例如imagef .available=2Gi)，描述kubelet在执行pod回收(如果该资源处于压力之下)时回收的最小资源量。 |
| --eviction-pressure-transition-period duration               | kubelet必须等待一段时间才能从驱逐压力状态过渡出来。(默认5m0s) |
| --eviction-soft mapStringString                              | 一组软驱逐阈值(例如memory.available<1.5Gi)，如果在相应的宽限期内达到该阈值，就会触发pod驱逐。 |
| --eviction-soft-grace-period mapStringString                 | 一组驱逐宽限期(例如memory.available=1m30s)，对应在触发pod驱逐之前，软驱逐阈值必须保持多长时间。 |
| --exit-on-lock-contention                                    | kubelet是否应该在锁文件争用时退出。                          |
| --experimental-allocatable-ignore-eviction                   | 当设置为“true”时，在计算节点可分配性时将忽略硬清除阈值。有关详细信息，请参见/docs/tasks/ administrator -cluster/reserve-compute-resources/。(默认= false) |
| --experimental-allowed-unsafe-sysctls stringSlice            | Comma-separated whitelist of unsafe sysctls or unsafe sysctl patterns (ending in *). Use these at your own risk. |
| --experimental-bootstrap-kubeconfig string                   | 弃用: use --bootstrap-kubeconfig                             |
| --experimental-check-node-capabilities-before-mount          | [实验]if set true, the kubelet will check the underlying node for required components (binaries, etc.) before performing the mount |
| --experimental-kernel-memcg-notification                     | If enabled, the kubelet will integrate with the kernel memcg notification to determine if memory eviction thresholds are crossed rather than polling. |
| --experimental-mounter-path string                           | [实验]Path of mounter binary. Leave empty to use the default mount. |
| --experimental-qos-reserved mapStringString                  | A set of ResourceName=Percentage (e.g. memory=50%) pairs that describe how pod resource requests are reserved at the QoS level. Currently only memory is supported. [default=none] |
| --fail-swap-on                                               | Makes the Kubelet fail to start if swap is enabled on the node. |
| --feature-gates mapStringBool                                | A set of key=value pairs that describe feature gates for alpha/experimental features. Options are: APIListChunking=true\|false (BETA - default=true) APIResponseCompression=true\|false (ALPHA - default=false) Accelerators=true\|false AdvancedAuditing=true\|false (BETA - default=true) AllAlpha=true\|false (ALPHA - default=false) AllowExtTrafficLocalEndpoints=true\|false AppArmor=true\|false (BETA - default=true) BlockVolume=true\|false (ALPHA - default=false) CPUManager=true\|false (BETA - default=true) CSIPersistentVolume=true\|false (ALPHA - default=false) CustomPodDNS=true\|false (ALPHA - default=false) CustomResourceValidation=true\|false (BETA - default=true) DebugContainers=true\|false  DevicePlugins=true\|false (ALPHA - default=false) DynamicKubeletConfig=true\|false (ALPHA - default=false) EnableEquivalenceClassCache=true\|false (ALPHA - default=false) ExpandPersistentVolumes=true\|false (ALPHA - default=false) ExperimentalCriticalPodAnnotation=true\|false (ALPHA - default=false) ExperimentalHostUserNamespaceDefaulting=true\|false (BETA - default=false) HugePages=true\|false (ALPHA - default=false) KubeletConfigFile=true\|false (ALPHA - default=false) LocalStorageCapacityIsolation=true\|false (ALPHA - default=false) MountContainers=true\|false (ALPHA - default=false) MountPropagation=true\|false (ALPHA - default=false) PVCProtection=true\|false (ALPHA - default=false) PersistentLocalVolumes=true\|false (ALPHA - default=false) PodPriority=true\|false (ALPHA - default=false) ReadOnlyAPIDataVolumes=true\|false ResourceLimitsPriorityFunction=true\|false (ALPHA - default=false) RotateKubeletClientCertificate=true\|false (BETA - default=true) RotateKubeletServerCertificate=true\|false (ALPHA - default=false) ServiceNodeExclusion=true\|false (ALPHA - default=false) ServiceProxyAllowExternalIPs=true\|false StreamingProxyRedirects=true\|false (BETA - default=true) SupportIPVSProxyMode=true\|false (ALPHA - default=false) TaintBasedEvictions=true\|false (BETA - default=true) TaintNodesByCondition=true\|false (BETA - default=true) VolumeScheduling=true\|false (ALPHA - default=false) VolumeSubpath=true\|false |
| --file-check-frequency duration                              | Duration between checking config files for new data (default 20s) |
| --global-housekeeping-interval duration                      | Interval between global housekeepings (default 1m0s)         |
| --google-json-key string                                     | The Google Cloud Platform Service Account JSON Key to use for authentication. |
| --hairpin-mode string                                        | How should the kubelet setup hairpin NAT. This allows endpoints of a Service to loadbalance back to themselves if they should try to access their own Service. Valid values are "promiscuous-bridge", "hairpin-veth" and "none". (default "promiscuous-bridge") |
| --healthz-bind-address 0.0.0.0                               | The IP address for the healthz server to serve on (set to 0.0.0.0 for all IPv4 interfaces and `::` for all IPv6 interfaces) (default 127.0.0.1) |
| --healthz-port int32                                         | The port of the localhost healthz endpoint (set to 0 to disable) (default 10248) |
| --host-ipc-sources stringSlice                               | Comma-separated list of sources from which the Kubelet allows pods to use the host ipc namespace. (default [*]) |
| --host-network-sources stringSlice                           | Comma-separated list of sources from which the Kubelet allows pods to use of host network. (default [*]) |
| --host-pid-sources stringSlice                               | Comma-separated list of sources from which the Kubelet allows pods to use the host pid namespace. (default [*]) |
| --hostname-override string                                   | If non-empty, will use this string as identification instead of the actual hostname. |
| --housekeeping-interval duration                             | Interval between container housekeepings (default 10s)       |
| --http-check-frequency duration                              | Duration between checking http for new data (default 20s)    |
| --image-gc-high-threshold int32                              | The percent of disk usage after which image garbage collection is always run. (default 85) |
| --image-gc-low-threshold int32                               | The percent of disk usage before which image garbage collection is never run. Lowest disk usage to garbage collect to. (default 80) |
| --image-pull-progress-deadline duration                      | If no pulling progress is made before this deadline, the image pulling will be cancelled. (default 1m0s) |
| --image-service-endpoint string                              | [实验]The endpoint of remote image service. If not specified, it will be the same with container-runtime-endpoint by default. Currently unix socket is supported on Linux, and tcp is supported on windows. |
| --init-config-dir string                                     | The Kubelet will look in this directory for the init configuration. The path may be absolute or relative; relative paths start at the Kubelet's current working directory. Omit this argument to use the built-in default configuration values. Presently, you must also enable the KubeletConfigFile feature gate to pass this flag. |
| --iptables-drop-bit int32                                    | The bit of the fwmark space to mark packets for dropping. Must be within the range [0, 31]. (default 15) |
| --iptables-masquerade-bit int32                              | The bit of the fwmark space to mark packets for SNAT. Must be within the range [0, 31]. Please match this parameter with corresponding parameter in kube-proxy. (default 14) |
| --kube-api-burst int32                                       | Burst to use while talking with kubernetes apiserver (default 10) |
| --kube-api-content-type string                               | Content type of requests sent to apiserver. (default "application/vnd.kubernetes.protobuf") |
| --kube-api-qps int32                                         | QPS to use while talking with kubernetes apiserver (default 5) |
| --kube-reserved mapStringString                              | A set of ResourceName=ResourceQuantity (e.g. cpu=200m,memory=500Mi,ephemeral-storage=1Gi,pid=1000) pairs that describe resources reserved for kubernetes system components. Currently cpu, memory, pid, and local ephemeral storage for root file system are supported. See http://kubernetes.io/docs/user-guide/compute-resources for more detail. [default=none] |
| --kube-reserved-cgroup string                                | Absolute name of the top level cgroup that is used to manage kubernetes components for which compute resources were reserved via '--kube-reserved' flag. Ex. '/kube-reserved'. [default=''] |
| --kubeconfig string                                          | Path to a kubeconfig file, specifying how to connect to the API server. Providing --kubeconfig enables API server mode, omitting --kubeconfig enables standalone mode. |
| --kubelet-cgroups string                                     | Optional absolute name of cgroups to create and run the Kubelet in. |
| --lock-file string                                           | The path to file for kubelet to use as a lock file.          |
| --log-backtrace-at traceLocation                             | when logging hits line file:N, emit a stack trace (default :0) |
| --log-cadvisor-usage                                         | Whether to log the usage of the cAdvisor container           |
| --log-dir string                                             | If non-empty, write log files in this directory              |
| --log-flush-frequency duration                               | Maximum number of seconds between log flushes (default 5s)   |
| --logtostderr                                                | log to standard error instead of files (default true)        |
| --machine-id-file string                                     | Comma-separated list of files to check for machine-id. Use the first one that exists. (default "/etc/machine-id,/var/lib/dbus/machine-id") |
| --make-iptables-util-chains                                  | If true, kubelet will ensure iptables utility rules are present on host. (default true) |
| --manifest-url string                                        | URL for accessing the container manifest                     |
| --manifest-url-header --manifest-url-header 'a:hello,b:again,c:world' --manifest-url-header 'b:beautiful' | Comma-separated list of HTTP headers to use when accessing the manifest URL. Multiple headers with the same name will be added in the same order provided. This flag can be repeatedly invoked. For example: --manifest-url-header 'a:hello,b:again,c:world' --manifest-url-header 'b:beautiful' |
| --max-open-files int                                         | Number of files that can be opened by Kubelet process. (default 1000000) |
| --max-pods int32                                             | Number of Pods that can run on this Kubelet. (default 110)   |
| --minimum-image-ttl-duration duration                        | Minimum age for an unused image before it is garbage collected. |
| --network-plugin string                                      | The name of the network plugin to be invoked for various events in kubelet/pod lifecycle |
| --network-plugin-mtu int32                                   | The MTU to be passed to the network plugin, to override the default. Set to 0 to use the default 1460 MTU. |
| --node-ip string                                             | IP address of the node. If set, kubelet will use this IP address for the node |
| --node-labels mapStringString                                | Labels to add when registering the node in the cluster.      |
| --node-status-update-frequency duration                      | Specifies how often kubelet posts node status to master. Note: be cautious when changing the constant, it must work with nodeMonitorGracePeriod in nodecontroller. (default 10s) |
| --oom-score-adj int32                                        | The oom-score-adj value for kubelet process. Values must be within the range [-1000, 1000] (default -999) |
| --pod-cidr string                                            | The CIDR to use for pod IP addresses, only used in standalone mode. |
| --pod-infra-container-image string                           | The image whose network/ipc namespaces containers in each pod will use. (default "k8s.gcr.io/pause:3.1") |
| --pod-manifest-path string                                   | Path to the directory containing pod manifest files to run, or the path to a single pod manifest file. Files starting with dots will be ignored. |
| --pods-per-core int32                                        | Number of Pods per core that can run on this Kubelet. The total number of Pods on this Kubelet cannot exceed max-pods, so max-pods will be used if this calculation results in a larger number of Pods allowed on the Kubelet. A value of 0 disables this limit. |
| --port int32                                                 | The port for the Kubelet to serve on. (default 10250)        |
| --protect-kernel-defaults                                    | Default kubelet behaviour for kernel tuning. If set, kubelet errors if any of kernel tunables is different than kubelet defaults. |
| --provider-id string                                         | Unique identifier for identifying the node in a machine database, i.e cloudprovider |
| --read-only-port int32                                       | The read-only port for the Kubelet to serve on with no authentication/authorization (set to 0 to disable) (default 10255) |
| --really-crash-for-testing                                   | If true, when panics occur crash. Intended for testing.      |
| --register-node                                              | Register the node with the apiserver. If --kubeconfig is not provided, this flag is irrelevant, as the Kubelet won't have an apiserver to register with. Default=true. (default true) |
| --register-with-taints []api.Taint                           | Register the node with the given list of taints (comma separated "=:"). No-op if register-node is false. |
| --registry-burst int32                                       | Maximum size of bursty pulls, temporarily allows pulls to burst to this number, while still not exceeding registry-qps. Only used if --registry-qps > 0 (default 10) |
| --registry-qps int32                                         | If > 0, limit registry pull QPS to this value.               |
| --resolv-conf string                                         | Resolver configuration file used as the basis for the container DNS resolution configuration. (default "/etc/resolv.conf") |
| --root-dir string                                            | Directory path for managing kubelet files (volume mounts,etc). (default "/var/lib/kubelet") |
| --rotate-certificates                                        | Auto rotate the kubelet client certificates by requesting new certificates from the kube-apiserver when the certificate expiration approaches. |
| --rotate-server-certificates                                 | Auto-request and rotate the kubelet serving certificates by requesting new certificates from the kube-apiserver when the certificate expiration approaches. Requires the RotateKubeletServerCertificate feature gate to be enabled, and approval of the submitted CertificateSigningRequest objects. |
| --runonce                                                    | If true, exit after spawning pods from local manifests or remote urls. Exclusive with --enable-server |
| --runtime-cgroups string                                     | Optional absolute name of cgroups to create and run the runtime in. |
| --runtime-request-timeout duration                           | Timeout of all runtime requests except long running request - pull, logs, exec and attach. When timeout exceeded, kubelet will cancel the request, throw out an error and retry later. (default 2m0s) |
| --seccomp-profile-root string                                | Directory path for seccomp profiles. (default "/var/lib/kubelet/seccomp") |
| --serialize-image-pulls                                      | Pull images one at a time. We recommend *not* changing the default value on nodes that run docker daemon with version < 1.9 or an Aufs storage backend. Issue #10959 has more details. (default true) |
| --stderrthreshold severity                                   | logs at or above this threshold go to stderr (default 2)     |
| --storage-driver-buffer-duration duration                    | Writes in the storage driver will be buffered for this duration, and committed to the non memory backends as a single transaction (default 1m0s) |
| --storage-driver-db string                                   | database name (default "cadvisor")                           |
| --storage-driver-host string                                 | database host:port (default "localhost:8086")                |
| --storage-driver-password string                             | database password (default "root")                           |
| --storage-driver-secure                                      | use secure connection with database                          |
| --storage-driver-table string                                | table name (default "stats")                                 |
| --storage-driver-user string                                 | database username (default "root")                           |
| --streaming-connection-idle-timeout duration                 | Maximum time a streaming connection can be idle before the connection is automatically closed. 0 indicates no timeout. Example: '5m' (default 4h0m0s) |
| --sync-frequency duration                                    | Max period between synchronizing running containers and config (default 1m0s) |
| --system-cgroups /                                           | Optional absolute name of cgroups in which to place all non-kernel processes that are not already inside a cgroup under /. Empty for no container. Rolling back the flag requires a reboot. |
| --system-reserved mapStringString                            | A set of ResourceName=ResourceQuantity (e.g. cpu=200m,memory=500Mi,ephemeral-storage=1Gi,pid=1000) pairs that describe resources reserved for non-kubernetes components. Currently only cpu, memory, and pid are supported. See http://kubernetes.io/docs/user-guide/compute-resources for more detail. [default=none] |
| --system-reserved-cgroup string                              | Absolute name of the top level cgroup that is used to manage non-kubernetes components for which compute resources were reserved via '--system-reserved' flag. Ex. '/system-reserved'. [default=''] |
| --tls-cert-file string                                       | File containing x509 Certificate used for serving HTTPS (with intermediate certs, if any, concatenated after server cert). If --tls-cert-file and --tls-private-key-file are not provided, a self-signed certificate and key are generated for the public address and saved to the directory passed to --cert-dir. |
| --tls-cipher-suites stringSlice                              | Comma-separated list of cipher suites for the server. If omitted, the default Go cipher suites will be used. Possible values: TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_RC4_128_SHA,TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_RC4_128_SHA,TLS_RSA_WITH_3DES_EDE_CBC_SHA,TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_128_GCM_SHA256,TLS_RSA_WITH_AES_256_CBC_SHA,TLS_RSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_RC4_128_SHA |
| --tls-private-key-file string                                | File containing x509 private key matching --tls-cert-file.   |
| -v, --v Level                                                | log level for V logs                                         |
| --version version[=true]                                     | Print version information and quit                           |
| --vmodule moduleSpec                                         | comma-separated list of pattern=N settings for file-filtered logging |
| --volume-plugin-dir string                                   | The full path of the directory in which to search for additional third party volume plugins (default "/usr/libexec/kubernetes/kubelet-plugins/volume/exec/") |
| --volume-stats-agg-period duration                           | Specifies interval for kubelet to calculate and cache the volume disk usage for all pods and volumes. |
| -h, --help                                                   | help for kubelet                                             |