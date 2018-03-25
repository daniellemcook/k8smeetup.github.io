
生成所有建立控制平面所需的PKI资产


### 概要


生成自签名 CA，为集群中的每个组件（包括节点）和客户端证书提供标识，供各种组件使用。

如果给定的的认证和私钥对都存在，kubeadm 跳过生成步骤而使用现有的文件。

Alpha 免责声明：此命令处于 Alpha 阶段。

```
kubeadm alpha phase certs all
```



 
### 例子

```
  # 生成所有建立控制平面所需的PKI资产，
  # 功能等同于 kubeadm init 所生成的。
  kubeadm alpha phase certs all
  
  # 使用配置文件的选项创建所有PKI资产。
  kubeadm alpha phase certs all --config masterconfiguration.yaml
```




### 选择项

```
      --apiserver-advertise-address string      可访问 API server 的IP地址，应用于 API server 服务证书
      --apiserver-cert-extra-sans stringSlice   应用于 API server 服务证书的可选的额外的别名。可以使IP地址和dns名
      --cert-dir string                         存储认证的路径（默认值 "/etc/kubernetes/pki"）
      --config string                           kubeadm 配置文件路径（警告: 使用配置文件处于实验阶段）
      --service-cidr string                     可替换的服务VIP的IP地址范围，源于加到 API Serever 的服务证书的内部 API server 的VIP（默认值 "10.96.0.0/12"）
      --service-dns-domain string               可替换的服务域，用于 API server 的服务证书（默认值 "cluster.local"）
```
