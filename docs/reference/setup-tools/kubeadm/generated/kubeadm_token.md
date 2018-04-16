
管理 bootstrap token


### 摘要




该命令用于管理 bootstrap token。 该命令是可选的，只在高级用例中需要。


简而言之，bootstrap token 用于在客户端和服务器端建立双向信任。
当一个客户端（如一个即将加入集群的节点）需要信任正在通信的服务器，这时可以使用带有 “签名” 的 bootstrap token。
Bootstrap token 也可以作为一种允许对 API 服务器进行短时间身份认证的方式（ bootstrap token 作为 API 服务器信任客户端的一种方式），如执行 TLS 初始化。



更确切地说，bootstrap token 是什么呢？
 - 它是 kube-system namespace 下的一个 Secret，其类型为 “bootstrap.kubernetes.io/token”。
 - bootstrap token 的形式为 “[a-z0-9]{6}.[a-z0-9]{16}”。 前一部分是 public token ID，而后一部分是
   Token Secret，并且它在任何情况下都必须保密。
 - Secret 必须以 “bootstrap-token-(token-id)” 的形式命名。


您可以在这里阅读更多关于 bootstrap token 的信息：
  https://kubernetes.io/docs/admin/bootstrap-tokens/

```
kubeadm token
```


### 选项

```
      --dry-run             是否启用 dry-run 模式
      --kubeconfig string   与集群通信时使用的 KubeConfig 文件(默认为 "/etc/kubernetes/admin.conf")
```

