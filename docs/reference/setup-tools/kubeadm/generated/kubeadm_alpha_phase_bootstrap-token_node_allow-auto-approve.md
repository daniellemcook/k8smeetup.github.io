
配置 RBAC 规则，以允许 csrapprover controller 自动批准来自节点 bootstrap token 的 CSR（证书签名请求）。


### 摘要



配置 RBAC 规则，以允许 csrapprover controller 自动批准由加入集群的节点生成的证书签名请求。
它（allow-auto-approve 命令）还为证书轮换配置 RBAC 规则（自动批准新证书）。


查看 TLS 初始化相关的在线文档，以了解更多详情。


Alpha 免责声明：该命令当前为 alpha。

```
kubeadm alpha phase bootstrap-token node allow-auto-approve
```


### 继承自父命令的选项

```
      --kubeconfig string   与集群通信时使用的 KubeConfig 文件(默认为 "/etc/kubernetes/admin.conf")
```

