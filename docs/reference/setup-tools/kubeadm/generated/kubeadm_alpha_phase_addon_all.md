将所有插件安装到 Kubernetes 集群

### 概要




通过 API 服务安装 kube-dns 和 kube-proxys 组件。
请注意，尽管部署了 DNS 服务器，但直到安装 CNI 后才会安排实施。

Alpha 版声明：此命令目前是 alpha 版。

```
kubeadm alpha phase addon all
```



### 例子

```
  # 通过 API 服务安装 kube-dns 和 kube-proxys 组件,
  # 在功能上等同于 kubeadm init 安装的内容。

  kubeadm alpha phase selfhosting from-staticpods
```



### 选项

```
      --apiserver-advertise-address string   可以访问的 API 服务的 IP 地址或 DNS 名称
      --apiserver-bind-port int32            可访问的 API 服务端口 （默认 6443）
      --config string                        kubeadm 的配置文件路径。 WARNING: 配置文件的使用是实验性的！      
      --feature-gates string                 一系列键值对，用来描述各种特性的功能开关。可选项为：
CoreDNS=true|false (ALPHA - default=false)
DynamicKubeletConfig=true|false (ALPHA - default=false)
HighAvailability=true|false (ALPHA - default=false)
SelfHosting=true|false (BETA - default=false)
StoreCertsInSecrets=true|false (ALPHA - default=false)
SupportIPVSProxyMode=true|false (ALPHA - default=false)
      --image-repository string              选择一个镜像仓库以从中提取 control plane 镜像 （默认 "k8s.gcr.io"）
      --kubeconfig string                    访问集群的 KubeConfig 文件 （默认 "/etc/kubernetes/admin.conf"）
      --kubernetes-version string            指定一个 Kubernetes 版本 （默认 "stable-1.8"）
      --pod-network-cidr string              用于 Pod 网络的 IP 地址范围
      --service-cidr string                  用于服务 VIP 的 IP 地址范围 （default "10.96.0.0/12"）
      --service-dns-domain string            服务的替代域名 （默认 "cluster.local"）
```
