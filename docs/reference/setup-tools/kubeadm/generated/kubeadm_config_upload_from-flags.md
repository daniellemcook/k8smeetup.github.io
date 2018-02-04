
首次使用参数（flag）来创建集群内部配置文件。


### 摘要




使用此命令，您可以上传配置到集群中的 ConfigMap，上传时使用的是 'kubeadm init' 中传入的相同的参数。
如果您使用 v1.7.x 或更低版本的 kubeadm 客户端，并设置某些参数来初始化您的集群，那么在使用 'kubeadm upgrade'
升级到 v1.8 之前，您需要使用同样的参数运行此命令。


配置位于 “kube-system” 名字空间内名为 “kubeadm-config” 的 ConfigMap 中。


```
kubeadm config upload from-flags
```


### 选项

```
      --apiserver-advertise-address string      API 服务器将通知监听的 IP 地址。 指定 '0.0.0.0' 以使用默认网络接口的地址。
      --apiserver-bind-port int32               API 服务器所绑定的端口 （默认为 6443）
      --apiserver-cert-extra-sans stringSlice   用于 API 服务器证书的可选附加主体别名（SAN）。 可以是 IP 地址和 DNS 名称。
      --cert-dir string                         保存和存储证书的路径。（默认为 “/etc/kubernetes/pki”）
      --feature-gates string                    一组描述多种特性的特性开关的键值对。 可选项有：
CoreDNS=true|false (ALPHA - 默认=false)
DynamicKubeletConfig=true|false (ALPHA - 默认=false)
HighAvailability=true|false (ALPHA - 默认=false)
SelfHosting=true|false (BETA - 默认=false)
StoreCertsInSecrets=true|false (ALPHA - 默认=false)
SupportIPVSProxyMode=true|false (ALPHA - 默认=false)
      --kubernetes-version string               为控制平面选择特定的 Kubernetes 版本（默认为 “stable-1.8”）
      --node-name string                        指定节点名称。
      --pod-network-cidr string                 为 pod 网络指定 IP 地址范围。 如果设置了该选项，控制平面会自动为每个节点分配 CIDR。
      --service-cidr string                     针对服务 VIP 使用的可选 IP 地址范围。 （默认为 “10.96.0.0/12”）
      --service-dns-domain string               针对服务使用的可选域，例如 “myorg.internal”。 （默认为 “cluster.local”）
      --token string                            用于在节点和 master 节点之间建立双向信任的 token。
      --token-ttl duration                      bootstrap token 自动删除前的持续时间。 如果设为 '0'，该 token 将永不过期（默认为 24h0m0s）
```


### 继承自父命令的选项

```
      --kubeconfig string   与集群通信时使用的 KubeConfig 文件(默认为 "/etc/kubernetes/admin.conf")
```

