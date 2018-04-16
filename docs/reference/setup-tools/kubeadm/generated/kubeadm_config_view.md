
查看存储在集群内部的 kubeadm 配置。


### 摘要




使用此命令，您可以查看集群中 kubeadm 配置所在的 ConfigMap。


配置位于 “kube-system” 名字空间内名为 “kubeadm-config” 的 ConfigMap 中。


```
kubeadm config view
```


### 继承自父命令的选项

```
      --kubeconfig string   与集群通信时使用的 KubeConfig 文件(默认为 "/etc/kubernetes/admin.conf")
```

