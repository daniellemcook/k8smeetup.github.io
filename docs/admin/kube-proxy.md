---
title: kube-proxy
notitle: true
cn-approvers:
- chentao1596
---
## kube-proxy




### 摘要



Kubernetes 网络代理运行在 node 上。它反映了 node 上 Kubernetes API 中定义的服务，并可以通过一组后端进行简单的 TCP、UDP 流转发或循环模式（round robin)）的 TCP、UDP 转发。目前，服务的集群 IP 和端口是通过 Docker-links 兼容的环境变量发现的，这些环境变量指定了服务代码打开的端口。有一个可选的 addon 为这些集群 IP 提供集群 DNS。用户必须使用 apiserver API 创建一个服务来配置代理。

```
kube-proxy
```


### 选项

```

      --azure-container-registry-config string       包含 Azure 容器注册配置信息的文件路径。

      --bind-address ip                              代理服务器提供服务的 IP 地址（设置为 0.0.0.0 代表在所有接口上提供服务）（默认为 0.0.0.0）

      --cleanup                                      如果为 true，退出时清理 iptables 和 ipvs 规则。

      --cluster-cidr string                          集群中 pod 的 CIDR 范围。配置后，从此范围外发送到服务集群 IP 的流量将被伪装，从 pod 发送到外部 LoadBalancer IP 的流量将被定向到相应的集群 IP

      --config string                                配置文件的路径。

      --config-sync-period duration                  从 apiserver 获取数据的频率。必须大于0。（默认 15m0s）

      --conntrack-max-per-core int32                 跟踪每个 CPU 核的 NAT 连接的最大数量（0 表示不配置并忽略 conntrack-min）。（默认 32768）

      --conntrack-min int32                          要分配的 conntrack 条目的最小数目, 不管 conntrack-max-per-core 是多少（设置 conntrack-max-per-core=0 会让该配置失效）。（默认 131072）

      --conntrack-tcp-timeout-close-wait duration    CLOSE_WAIT 状态下 TCP 连接的 NAT 超时时间（默认 1h0m0s）

      --conntrack-tcp-timeout-established duration   已建立的 TCP 连接的空闲超时时间（0 表示不配置）（默认 24h0m0s）

      --feature-gates mapStringBool                  一组 key=value 对，用于描述 “alpha/实验特性” 的开关。可选项包括：
APIListChunking=true|false (ALPHA - default=false)
APIResponseCompression=true|false (ALPHA - default=false)
Accelerators=true|false (ALPHA - default=false)
AdvancedAuditing=true|false (BETA - default=true)
AllAlpha=true|false (ALPHA - default=false)
AllowExtTrafficLocalEndpoints=true|false (default=true)
AppArmor=true|false (BETA - default=true)
CPUManager=true|false (ALPHA - default=false)
CustomResourceValidation=true|false (ALPHA - default=false)
DebugContainers=true|false (ALPHA - default=false)
DevicePlugins=true|false (ALPHA - default=false)
DynamicKubeletConfig=true|false (ALPHA - default=false)
EnableEquivalenceClassCache=true|false (ALPHA - default=false)
ExpandPersistentVolumes=true|false (ALPHA - default=false)
ExperimentalCriticalPodAnnotation=true|false (ALPHA - default=false)
ExperimentalHostUserNamespaceDefaulting=true|false (BETA - default=false)
HugePages=true|false (ALPHA - default=false)
Initializers=true|false (ALPHA - default=false)
KubeletConfigFile=true|false (ALPHA - default=false)
LocalStorageCapacityIsolation=true|false (ALPHA - default=false)
MountPropagation=true|false (ALPHA - default=false)
PersistentLocalVolumes=true|false (ALPHA - default=false)
PodPriority=true|false (ALPHA - default=false)
RotateKubeletClientCertificate=true|false (BETA - default=true)
RotateKubeletServerCertificate=true|false (ALPHA - default=false)
StreamingProxyRedirects=true|false (BETA - default=true)
SupportIPVSProxyMode=true|false (ALPHA - default=false)
TaintBasedEvictions=true|false (ALPHA - default=false)
TaintNodesByCondition=true|false (ALPHA - default=false)

      --google-json-key string                       用于身份认证的 Google 云平台服务账户 JSON 密钥。

      --healthz-bind-address ip                      健康检查服务提供服务的 IP 地址和端口（设置为 0.0.0.0 代表在所有接口上提供服务）（默认 0.0.0.0:10256）

      --healthz-port int32                           绑定健康检查服务的端口。使用 0 表示禁用服务（默认 10256）

      --hostname-override string                     如果非空，将使用此字符串作为主机标识，而不是实际的主机名。

      --iptables-masquerade-bit int32                如果使用纯 iptables 代理，这个 fwmark 空间的位用来标记需要 SNAT 的数据包。必须在范围 [0，31] 以内。（默认 14）

      --iptables-min-sync-period duration            随着 endpoint 和 service 的变化，可以刷新 iptables 规则的最小间隔（例如 '5s'，'1m'，'2h22m'）。

      --iptables-sync-period duration                iptables 规则刷新的最大间隔（例如 '5s'，'1m'，'2h22m'）。必须大于 0 （默认 30s）

      --ipvs-min-sync-period duration                随着 endpoint 和 service 的变化，可以刷新 ipvs 规则的最小间隔（例如 '5s'，'1m'，'2h22m'）（例如 '5s'，'1m'，'2h22m'）。

      --ipvs-scheduler string                        代理模式为 ipvs 时的 ipvs 调度类型

      --ipvs-sync-period duration                    ipvs 规则刷新的最大间隔（例如 '5s'，'1m'，'2h22m'）。必须大于 0。

      --kube-api-burst int                           与 kubernetes apiserver 通信时使用的 Burst（默认 10）

      --kube-api-content-type string                 发送到 apiserver 的内容类型。（默认 "application/vnd.kubernetes.protobuf"）

      --kube-api-qps float32                         与 kubernetes apiserver 通信时使用的 QPS（默认 5）

      --kubeconfig string                            包含鉴权信息的 kubeconfig 文件路径（master 的位置由 master 标志来设定）

      --masquerade-all                               如果使用纯 iptables 代理，SNAT 所有流量，将它们改为通过服务集群 IP 发送 (并不常见)

      --master string                                Kubernetes API 服务的地址（会覆盖 kubeconfig 文件中设定的值）

      --metrics-bind-address ip                      度量服务提供服务的 IP 地址和端口（设置为 0.0.0.0 代表在所有接口上提供服务）（默认 127.0.0.1:10249）

      --oom-score-adj int32                          kube-proxy 进程的 oom-score-adj 值。必须在 [-1000, 1000] 范围内（默认 -999）

      --profiling                                    如果为 true，则能够通过 /debug/pprof 处理器上的 web 接口执行 profiling。

      --proxy-mode ProxyMode                         使用的代理模式：'userspace'（旧一些的方式） 或者 'iptables'（更快的方式）或者 'ipvs'（实验性的方式）。如果为空，将使用最佳可用的代理（目前是 iptables）。如果选择了 iptables 代理，但是系统的内核或 iptables 版本不够，不管怎么样都会回到使用 userspace 代理。

      --proxy-port-range port-range                  为了代理服务流量而使用的主机端口的范围（开始端口-结束端口，闭区间）。如果没有指定（0-0），则随机选择端口。

      --udp-timeout duration                         空闲的 UDP 连接将保持打开状态的时长(例如 “250 ms”、“2s”)。必须大于0。只适用于 proxy-mode=userspace（默认 250 ms）

      --version version[=true]                       打印版本信息并退出

      --write-config-to string                       如果设置了，则将默认配置值写入该文件并退出。
```

###### Auto generated by spf13/cobra on 27-Sep-2017
