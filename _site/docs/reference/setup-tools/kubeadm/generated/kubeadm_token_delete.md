
在服务器上删除 bootstrap token


### 摘要




该命令会为您删除给定的 bootstrap token。


其中 [token-value] 是要删除的形式为 “[a-z0-9]{6}.[a-z0-9]{16}” 的完整 Token，
或形式为 “[a-z0-9]{6}” 的 Token ID。


```
kubeadm token delete [token-value]
```


### 继承自父命令的选项

```
      --dry-run             是否启用 dry-run 模式
      --kubeconfig string   与集群通信时使用的 KubeConfig 文件(默认为 "/etc/kubernetes/admin.conf")
```

