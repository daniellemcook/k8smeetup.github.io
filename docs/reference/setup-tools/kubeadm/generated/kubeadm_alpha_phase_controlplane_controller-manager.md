生成 controller-manager 静态 Pod 清单。

### 概要





为 controller-manager 生成静态 Pod 清单，并将其保存进 /etc/kubernetes/manifests/kube-controller-manager.yaml 文件。



### 选项

```
      --cert-dir string             证书存储路径 （默认 "/etc/kubernetes/pki"）
      --config string               kubeadm 的配置文件路径 （WARNING: 配置文件的使用是实验性的）
      --kubernetes-version string   指定一个 Kubernetes 版本 （默认 "stable-1.8"）
      --pod-network-cidr string     用于 Pod 网络的 IP 地址范围
```
