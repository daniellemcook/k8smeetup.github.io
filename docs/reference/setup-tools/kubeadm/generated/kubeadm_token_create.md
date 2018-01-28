
在服务器上创建 bootstrap token。


### 摘要




该命令会为您创建 bootstrap token。
您可以指定该 token 的用法，包括 “生存时间” 和可选的人性化描述。



其中 [token] 是要写入的实际 token。
它应该是一个安全生成的随机 token，其形式为 “[a-z0-9]{6}.[a-z0-9]{16}”。
如果没有给定 [token]，kubeadm 会生成一个随机 token。


```
kubeadm token create [token]
```


### 选项

```
      --description string   关于该 token 如何使用的人性化描述
      --groups stringSlice   当用于认证时，该 token 会以这些额外组身份进行认证。 必须和 “system:bootstrappers:[a-z0-9:-]{0,255}[a-z0-9]” 的格式匹配。（默认为 [system:bootstrappers:kubeadm:default-node-token]）
      --print-join-command   输出使用 token 加入集群所需的完整 'kubeadm join' 参数，而不是只输出 token。
      --ttl duration         token 自动删除前的持续时间（例如 1s、 2m 或 3h）。 如果设为 '0'，该 token 将永不过期（默认为 24h0m0s）
      --usages stringSlice   描述该 token 允许的使用方式。 您可以传入 --usages 多次，或者提供一个逗号分隔的选项列表。 有效的选项： [signing,authentication]。（默认为[signing,authentication]）
```


### 继承自父命令的选项

```
      --dry-run             是否启用 dry-run 模式
      --kubeconfig string   与集群通信时使用的 KubeConfig 文件(默认为 "/etc/kubernetes/admin.conf")
```
