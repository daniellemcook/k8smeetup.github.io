---
title: cloud-controller-manager
notitle: true
---
## cloud-controller-manager



### 大纲


云控制器管理器（cloud controller manager）是一个 Kubernetes 附带的，嵌入了特定云服务控制循环逻辑的守护进程。

```
cloud-controller-manager
```


### 选项

```

      --address ip                               提供服务的 IP 地址（设置为 0.0.0.0 表示使用所有接口）。 （默认为 0.0.0.0）
      
      --allocate-node-cidrs                      应该在云服务提供商（cloud provider）上为 Pod 分配和设置的 CIDR。
      
      --azure-container-registry-config string   包含 Azure 容器仓库配置信息的文件路径。
      
      --cloud-config string                      云服务提供商（cloud provider）配置文件路径。空字符串表示无配置文件。
      
      --cloud-provider string                    云服务的提供商名。不能为空。
      
      --cluster-cidr string                      集群中 Pod 的 CIDR 范围。
      
      --cluster-name string                      集群实例前缀。 （默认为 "kubernetes"）
      
      --configure-cloud-routes                   CIDR 是否应该由云服务提供商配置的 allocate-node-cidrs 分配。 （默认为 true）
      
      --contention-profiling                     如果启用了性能分析（profiling），则启用锁竞争分析（lock contention profiling）。
      
      --controller-start-interval duration       启动 controller manager 的时间间隔。
      
--feature-gates mapStringBool              一组描述 alpha/experimental 特性的 feature gate 的 key=value 键值对。选项包括：
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

      --google-json-key string                   用于认证的 Google Cloud Platform Service Account JSON Key。
      
      --kube-api-burst int32                     和 kubernetes apiserver 通信时使用的 burst 值。 （默认为 30）
      
      --kube-api-content-type string             发送给 apiserver 的请求的 content type。 （默认为 "application/vnd.kubernetes.protobuf"）
      
      --kube-api-qps float32                     和 kubernetes apiserver 通信时使用的 QPS 值。 （默认为 20）
      
      --kubeconfig string                        包含认证信息及主节点连接信息的 kubeconfig 文件路径。
      
      --leader-elect                             在执行主要控制循环逻辑前，启动 leader election 客户端并获得 leadership。请在需要副本化运行组件以实现高可用时启用此选项。 （默认为 true）
       
      --leader-elect-lease-duration duration     从发现 leadership 更新起，到非 leader 候选者尝试获取领导权（但并没有实际更新 leader）需要等待的时间。这实际上是一个 leader 在被另一个候选者替代之前停止所需要花费的时间。这个参数只在启用了 leader election 时生效。 （默认为 15s）
      
      --leader-elect-renew-deadline duration     在停止领导前，活动主节点尝试更新 leadership 地位的时间间隔。此选项必须小于等于 lease duration。这个参数只在启用了 leader election 时生效。  （默认为 10s）
      
      --leader-elect-resource-lock endpoints     在 leader election 过程中充当锁使用的资源对象类型。支持的选项有 endpoints（默认）和 `configmap`。 （默认为 "endpoints"）
      
      --leader-elect-retry-period duration       客户端在尝试请求及更新 leadership 时需要等待的时间间隔。 这个参数只在启用了 leader election 时生效。 （默认为 2s）
      
      --master string                            Kubernetes API server  地址（将覆盖 kubeconfig 中的值）。
      
      --min-resync-period duration               Reflector 中的 resync 周期将为 MinResyncPeriod 到 2*MinResyncPeriod 之间的随机值。 （默认为 12h0m0s）
      
      --node-monitor-period duration             NodeController 中 NodeStatus 的同步时间周期。 （默认为 5s）
      
      --node-status-update-frequency duration    定义控制器更新节点状态的周期。（默认为 5m0s）
      
      --port int32                               cloud-controller-manager http 服务监听的端口。（默认为 10253）
      
      --profiling                                通过 host:port/debug/pprof/ web 接口启用性能分析（profiling）。 （默认为 true）
      
      --route-reconciliation-period duration     云服务提供商为节点创建的路由的调节周期。
      
      --use-service-account-credentials          如果为 true，则为每个控制器使用单独的 service account 凭据。
      
      --version version[=true]                   打印版本信息后退出。
```

###### Auto generated by spf13/cobra on 27-Sep-2017
