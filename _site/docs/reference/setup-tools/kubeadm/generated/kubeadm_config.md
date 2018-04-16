
管理 kubeadm 集群的配置，这些配置持久化保存在集群的一个 ConfigMap 中。


### 摘要




在 kube-system 名字空间里有一个名为 “kubeadm-config” 的 ConfigMap，kubeadm 用它来存储集群的内部配置。
kubeadm CLI v1.8.0 以上的版本会用 'kubeadm init' 中使用的配置来自动创建这一 ConfigMap，但是如果您使用
kubeadm v1.7.x 或更低的版本来初始化集群，那么您必须使用 'config upload' 命令来创建这一 ConfigMap。
这是必需的，以便 'kubeadm upgrade' 能够正确配置升级后的集群。


```
kubeadm config
```


### 选项

```
      --kubeconfig string   与集群通信时使用的 KubeConfig 文件(默认为 "/etc/kubernetes/admin.conf")
```

