---
title: kubefed
notitle: true
---
## kubefed


kubefed 对 Kubernetes 集群集合进行管理


### 概要



kubefed 对 Kubernetes 集群集合进行管理


更多信息请见[https://github.com/kubernetes/kubernetes](https://github.com/kubernetes/kubernetes)。

```
kubefed
```


### 选项


```
      --alsologtostderr                         标准错误日志及文件日志
      --as string                               模仿操作的 Username
      --as-group stringArray                    模仿操作的 Group, 这个标签可以重复用于指定多个 group
      --certificate-authority string            授权机构发布的证书文件路径
      --client-certificate string               TLS 客户端证书文件路径
      --client-key string                       TLS 客户端密钥文件路径
      --cloud-provider-gce-lb-src-cidrs cidrs   GCE 防火墙中为负载均衡代理和健康检查开放的 CIDRS (默认 209.85.152.0/22,209.85.204.0/22,130.211.0.0/22,35.191.0.0/16)
      --cluster string                          kubeconfig 使用的主机名
      --context string                          kubeconfig 使用的context名
      --insecure-skip-tls-verify                如果设置为真，服务器证书将不会被验证，这将会使您的HTTPS连接不安全
      --kubeconfig string                       用于 CLI 请求的 kubeconfig 文件
      --log-backtrace-at traceLocation          when logging hits line file:N, emit a stack trace (default :0)
      --log-dir string                          如果非空，在此路径下写入日志文件
      --log-flush-frequency duration            日志最大清理间隔，单位为秒（默认 5s）
      --logtostderr                             标准 error 而不是文件的日志 (默认 true)
      --match-server-version                    是否需要服务器和客户端版本同步
  -n, --namespace string                        如果有，指定 CLI 请求作用的 namespace 范围
      --password string                         API 服务器的基础验证密码
      --request-timeout string                  对于单个服务器请求的最长等待时间。非零值需要指定相应的时间单位（例如：1s, 2m, 3h）如果为零，表示不设定请求超时。 (默认 "0")
  -s, --server string                           Kubernetes API 服务器的地址和端口
      --stderrthreshold severity                超过此阈值的 stderrlogs 将被发送到stderr (默认 2)
      --token string                            不记名 token，用于向API服务器进行身份验证
      --user string                             kubeconfig 用户名
      --username string                         API 服务器基础验证时使用的用户名
  -v, --v Level                                 V 日志的日志级别
      --vmodule moduleSpec                      comma-separated list of pattern=N settings for file-filtered logging
```


### 请参阅
* [kubefed init](kubefed_init.md)	 - federation 层的初始化
* [kubefed join](kubefed_join.md)	 - 向 federation 新增一个集群
* [kubefed options](kubefed_options.md)	 - 所有命令的标志位清单
* [kubefed unjoin](kubefed_unjoin.md)	 - 从 federation 中移出一个集群
* [kubefed version](kubefed_version.md)	 - 打印客户端和服务器的版本信息


###### spf13/cobra 自动生成于 2017/07/30
