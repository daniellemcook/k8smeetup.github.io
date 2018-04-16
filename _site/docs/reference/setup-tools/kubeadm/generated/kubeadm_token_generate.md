
生成并输出一个 bootstrap token，但不在服务器上创建它。


### 摘要




该命令会输出一个随机生成的 bootstrap token，该 token 可被用于 “init” 和 “join” 命令。


不是必须使用该命令来生成 token，您也可以自己来生成，只要保证 token 格式为 “[a-z0-9]{6}.[a-z0-9]{16}”。
该命令的提供是为了方便生成给定格式的 token。


您也可以在不指定 token 的情况下使用 “kubeadm init” 命令，它将为您生成并输出一个 token。


```
kubeadm token generate
```


### 继承自父命令的选项

```
      --dry-run             是否启用 dry-run 模式
      --kubeconfig string   与集群通信时使用的 KubeConfig 文件(默认为 "/etc/kubernetes/admin.conf")
```

