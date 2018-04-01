---
approvers:
- bowei
- zihongz
title:  调试 DNS 解析
cn-approvers:
- chentao1596
---


{% capture overview %}

此页面提供有关诊断 DNS 问题的提示。
{% endcapture %}

{% capture prerequisites %}
* {% include task-tutorial-prereqs.md %}

* Kubernetes 版本 1.6 及以上。
* 该集群必须配置为使用 `kube-dns` 插件。
{% endcapture %}

{% capture steps %}


### 创建一个简单的 Pod 来用作测试环境


使用以下内容创建一个名为 busybox.yaml 的文件：

{% include code.html language="yaml" file="busybox.yaml" ghlink="/docs/tasks/administer-cluster/busybox.yaml" %}


然后使用此文件创建一个 pod 并验证其状态：

```shell
$ kubectl create -f busybox.yaml
pod "busybox" created

$ kubectl get pods busybox
NAME      READY     STATUS    RESTARTS   AGE
busybox   1/1       Running   0          <some-time>
```


一旦该 pod 运行，您就可以在环境中执行 `nslookup`。如果您看到如下所示的内容，则 DNS 工作正常。

```shell
$ kubectl exec -ti busybox -- nslookup kubernetes.default
Server:    10.0.0.10
Address 1: 10.0.0.10

Name:      kubernetes.default
Address 1: 10.0.0.1
```


如果 `nslookup` 命令失败，请检查以下内容：


### 首先检查本地 DNS 配置


看一看 resolv.conf 文件。（有关更多信息，请参阅 [从节点继承 DNS](#inheriting-dns-from-the-node) 和 下面的 [已知问题](#已知问题)）

```shell
$ kubectl exec busybox cat /etc/resolv.conf
```


验证搜索路径和名称服务器是否设置如下（请注意，搜索路径可能因不同的云提供商而异）：

```
search default.svc.cluster.local svc.cluster.local cluster.local google.internal c.gce_project_id.internal
nameserver 10.0.0.10
options ndots:5
```


以下错误表明 kube-dns 附加组件或相关服务存在问题：

```
$ kubectl exec -ti busybox -- nslookup kubernetes.default
Server:    10.0.0.10
Address 1: 10.0.0.10

nslookup: can't resolve 'kubernetes.default'
```


或者

```
$ kubectl exec -ti busybox -- nslookup kubernetes.default
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

nslookup: can't resolve 'kubernetes.default'
```


### 检查 DNS pod 是否正在运行中


使用 `kubectl get pods` 命令验证 DNS pod 是否正在运行中。

```shell
$ kubectl get pods --namespace=kube-system -l k8s-app=kube-dns
NAME                    READY     STATUS    RESTARTS   AGE
...
kube-dns-v19-ezo1y      3/3       Running   0           1h
...
```


如果您看到没有 pod 正在运行中，或者 pod 已失败/已完成，那么在当前环境中，默认情况下可能不会部署 DNS 插件，您将不得不手动部署它。


### 检查 DNS pod 中的错误


使用 `kubectl logs` 命令查看 DNS 守护程序的日志。

```shell
$ kubectl logs --namespace=kube-system $(kubectl get pods --namespace=kube-system -l k8s-app=kube-dns -o name) -c kubedns
$ kubectl logs --namespace=kube-system $(kubectl get pods --namespace=kube-system -l k8s-app=kube-dns -o name) -c dnsmasq
$ kubectl logs --namespace=kube-system $(kubectl get pods --namespace=kube-system -l k8s-app=kube-dns -o name) -c sidecar
```


看看有没有可疑的日志。字母 '`W`'、'`E`'、'`F`' 表示警告、错误和失败。请搜索具有这些日志级别的条目，并使用 [kubernetes 问题](https://github.com/kubernetes/kubernetes/issues) 来报告意外错误。


### DNS服务起来了吗？


通过使用 `kubectl get service` 命令验证 DNS 服务已启动。

```shell
$ kubectl get svc --namespace=kube-system
NAME          CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
...
kube-dns      10.0.0.10      <none>        53/UDP,53/TCP        1h
...
```


如果您已经创建了该服务，或者应该在默认情况下创建它，但它没有出现，请参阅 [调试服务](/docs/tasks/debug-application-cluster/debug-service/) 以获取更多信息。


### DNS endpoints 是否暴露？


您可以使用 `kubectl get endpoints` 命令验证是否暴露了了 DNS endpoints。

```shell
$ kubectl get ep kube-dns --namespace=kube-system
NAME       ENDPOINTS                       AGE
kube-dns   10.180.3.17:53,10.180.3.17:53    1h
```


如果您没有看到 endpoints，请参阅 [调试服务](/docs/tasks/debug-application-cluster/debug-service/) 文档中的 endpoints 部分 。


有关其他 Kubernetes DNS 示例，请参阅 Kubernetes GitHub 仓库中的 [cluster-dns 示例](https://github.com/kubernetes/examples/tree/master/staging/cluster-dns)。


## 已知问题


Kubernetes 安装不会将节点的 resolv.conf 文件配置为默认使用集群 DNS，因为该过程本身就是发行版的。最终可能会这么实现。


Linux 的 libc 不可能摆脱（[见 2005 年的这个 bug](https://bugzilla.redhat.com/show_bug.cgi?id=168253)）只有 3 个 DNS `nameserver` 记录和 6 个 DNS `search` 记录的限制。Kubernetes 需要消耗 1 个 `nameserver` 记录和 3 条 `search` 记录。这意味着如果本地安装已经使用了 3 个 `nameserver` 或使用了多于 3 条 `search`，那么其中一些设置将会丢失。作为部分解决方法，节点可以运行 `dnsmasq`，它将提供更多 `nameserver` 条目，但没有更多的 `search` 条目。您也可以使用 kubelet `--resolv-conf` 标志。


如果您使用 Alpine 3.3 或更低版本作为您的基本镜像，由于 Alpine 的某些已知问题，DNS 可能无法正常工作。点击 [这里](https://github.com/kubernetes/kubernetes/issues/30215) 查看更多信息。


## Kubernetes Federation（多区域支持）


版本 1.3 引入了用于多站点 Kubernetes 安装的集群 Federation 支持。这需要对 Kubernetes 集群 DNS 服务器处理 DNS 查询的方式进行一些小的（向后兼容）更改，以便于查找 Federation 服务（跨多个 Kubernetes 集群）。有关 Cluster Federation 和多站点支持的更多详细信息，请参见 [Cluster Federation 管理员指南](/docs/concepts/cluster-administration/federation/)。


## 参考


- [Service 和 Pod 的 DNS](/docs/concepts/services-networking/dns-pod-service/)
- [DNS 集群插件的文档](http://releases.k8s.io/{{page.githubbranch}}/cluster/addons/dns/README.md)


## 下一步是什么
- [在集群中自动调节 DNS 服务](/docs/tasks/administer-cluster/dns-horizontal-autoscaling/)。

{% endcapture %}

{% include templates/task.md %}