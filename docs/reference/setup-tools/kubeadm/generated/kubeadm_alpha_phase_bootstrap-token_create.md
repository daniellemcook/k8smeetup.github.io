
创建一个 bootstrap token 用于加入节点


### 摘要



创建一个 bootstrap token。如果没有给 token，kubeadm 会生成一个随机 token。

或者，您可以使用 kubeadm token。

alpha 声明：这个命令是当前的 alpha。

```
kubeadm alpha phase bootstrap-token create
``` 



### 选项

```
      --cert-dir string      存储证书的路径 (默认 "/etc/kubernetes/pki")
      --config string        kubeadm配置文件的路径 （警告：配置文件的使用是实验性的）
      --description string   关于如何使用该token的人性化描述。 （默认是 "由'kubeadm init'生成的默认bootstrap token。"）
      --groups stringSlice   在认证时 token 将会用来认证的额外组。必须匹配 "system:bootstrappers:[a-z0-9:-]{0,255}[a-z0-9]" (默认 [system:bootstrappers:kubeadm:default-node-token])
      --skip-token-print     跳过 bootstrap token 的打印
      --token string         用于在节点和 masters 之间建立双向信任的 token
      --ttl duration          token 被自动删除前的持续时间（例如，1s, 2m, 3h）。 如果设置为 '0', token 永远不会过期 (默认 24h0m0s)
      --usages stringSlice   介绍可以使用这个 token的方式。您可以多次传递 --usages 或提供逗号分隔的选项列表。 有效的选项: [signing,authentication] (默认 [signing,authentication])
```




### 继承自父命令的选项

```
      --kubeconfig string   与集群通信时使用的 KubeConfig 文件(默认为 "/etc/kubernetes/admin.conf")
```

