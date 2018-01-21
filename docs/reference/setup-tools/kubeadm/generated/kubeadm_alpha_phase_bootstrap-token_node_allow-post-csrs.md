
配置 RBAC，来允许节点 bootstrap token 发送 CSR（证书签名请求），以使得节点获得长期证书凭证。


### 摘要



配置 RBAC 规则，以允许节点 bootstrap token 发送证书签名请求，
从而使加入集群的节点能够请求长期证书凭证。


查看 TLS 初始化相关的在线文档，以了解更多详情。


Alpha 免责声明：该命令当前为 alpha。

```
kubeadm alpha phase bootstrap-token node allow-post-csrs
```


### 继承自父命令的选项

```
      --kubeconfig string   与集群通信时使用的 KubeConfig 文件(默认为 "/etc/kubernetes/admin.conf")
```

