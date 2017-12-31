---
approvers:
- aveshagarwal
- eparis
- thockin
cn-approvers:
- lichuqiang
title: Fedora (单节点)
---


* TOC
{:toc}


## 前提

1. 您需要 2 台或更多安装了 Fedora 的机器。 这些机器可以是裸金属机或虚拟机。


## 操作指南

本文是 Fedora 的入门指南。 这是一种手动配置，所以您需要理解所有底层包/服务/端口等等。


本指南只能使得一个 node（以前的 minion）运行。 如要运行多个 node，需要在 Kubernetes 之外进行功能性的
[网络配置](/docs/concepts/cluster-administration/networking/)，尽管这一过程中显然还需要额外的 Kubernetes 配置。


Kubernetes 包提供了一些服务：kube-apiserver、 kube-scheduler、 kube-controller-manager、 kubelet 和 kube-proxy。 这些服务由 systemd 管理，其配置位于一个中心位置： `/etc/kubernetes`。
我们会在不同的主机间对这些服务进行拆分。 第一台主机——fed-master，会作为 Kubernetes master。
这台主机将会运行 kube-apiserver、 kube-controller-manager 和 kube-scheduler。 此外，master
还会运行 _etcd_ （如果 _etcd_ 运行在其他主机上则不需要，但本指南假设 _etcd_ 和 Kubernetes master
运行在同一主机上）。 余下的主机——fed-node 会作为 node，并运行 kubelet、 proxy 以及 docker。


**系统信息：**

主机：

```conf
fed-master = 192.168.121.9
fed-node = 192.168.121.65
```


**准备主机：**

* 在所有主机上安装 Kubernetes - fed-{master，node}。 这一过程中还将安装 docker。 还会在 fed-master 上安装 etcd。 本指南已经使用 Kubernetes-0.18 及以上版本进行了测试。

* 为了在带有 RHEL 7.2 的 AWS EC2 主机上运行，您需要通过以下操作来为 yum 启用 “extras” 仓库：
编辑 `/etc/yum.repos.d/redhat-rhui.repo` 并将 “extras” 的 `enable=0` 改为 `enable=1`。

```shell
dnf -y install kubernetes
```


* 安装 etcd

```shell
dnf -y install etcd
```


* 将 master 和 node 加入到所有机器的 `/etc/hosts` 文件中（如果主机名已经在 DNS 中则不需要)。
通过使用类似 ping 的功能来确保 fed-master 和 fed-node 间通信正常。

```shell
echo "192.168.121.9    fed-master
192.168.121.65    fed-node" >> /etc/hosts
```


* 编辑 `/etc/kubernetes/config` (在所有主机上应该相同) 来设置 master 服务器的名称：


```shell
# 以逗号分隔的 etcd 集群 node 列表
KUBE_MASTER="--master=http://fed-master:8080"
```


* 由于 docker 在有其他防火墙规则管理器的情况下无法正常工作，请禁用 master 和 node 的防火墙。
请注意，iptables-services 在默认安装的 fedora 服务器上是不存在的。

```shell
systemctl disable iptables-services firewalld
systemctl stop iptables-services firewalld
```


**在 master 上配置 Kubernetes 服务。**


* 编辑 `/etc/kubernetes/apiserver` 为如下形式。 其中 service-cluster-ip-range IP 地址必须是一个未使用的地址块，不在其他任何地方使用。 这些地址不需要进行任何路由或分配。


```shell
# 本地服务器监听的地址。
KUBE_API_ADDRESS="--address=0.0.0.0"

# 以逗号分隔的 etcd 集群 node 列表
KUBE_ETCD_SERVERS="--etcd-servers=http://127.0.0.1:2379"

# 用于 service 的地址范围
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"

# 添加您自己的参数！
KUBE_API_ARGS=""
```


* 编辑 `/etc/etcd/etcd.conf`，使 etcd 监听在所有可用 IP 上，而不是 127.0.0.1。 如果没有进行该操作，
您可能会看到类似 "connection refused" 的错误。

```shell
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
```


* 在 master 上启动相应的服务：

```shell
for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler; do
    systemctl restart $SERVICES
    systemctl enable $SERVICES
    systemctl status $SERVICES
done
```


* 添加 node：


* 在 Kubernetes master 节点上创建下列 node.json 文件：

```json
{
    "apiVersion": "v1",
    "kind": "Node",
    "metadata": {
        "name": "fed-node",
        "labels":{ "name": "fed-node-label"}
    },
    "spec": {
        "externalID": "fed-node"
    }
}
```


现在通过运行以下命令在您的 Kubernetes 集群内部创建一个 node 对象：

```shell
$ kubectl create -f ./node.json

$ kubectl get nodes
NAME            STATUS        AGE      VERSION
fed-node        Unknown       4h
```


请注意，上面的命令只是内部创建了一个 _fed-node_ 节点的代表。 它并不提供真实的 _fed-node_。
另外，我们假定 _fed-node_ （按照 `name` 中指定的）能够在 Kubernetes master 节点上解析并访问。
本指南下面会讨论如何提供一个 Kubernetes node(fed-node)。


**在 node 上 配置 Kubernetes 服务。**


***我们需要在 node 上配置 kubelet。***

* 编辑 `/etc/kubernetes/kubelet` 为如下形式：



```shell
###
# Kubernetes kubelet (node) 配置

# 服务器监听的地址信息 (针对所有网络接口，设为 0.0.0.0 或 "")
KUBELET_ADDRESS="--address=0.0.0.0"

# 您可以将该参数设置为空，来使用实际的主机名
KUBELET_HOSTNAME="--hostname-override=fed-node"

# api-server 的位置
KUBELET_API_SERVER="--api-servers=http://fed-master:8080"

# 添加您自己的参数！
#KUBELET_ARGS=""
```


* 在 node (fed-node)上启动相应的服务：

```shell
for SERVICES in kube-proxy kubelet docker; do 
    systemctl restart $SERVICES
    systemctl enable $SERVICES
    systemctl status $SERVICES 
done
```


* 现在检查集群，确保在 fed-master 上能够看到 fed-node，且其状态变为 _Ready_。

```shell
kubectl get nodes
NAME            STATUS      AGE      VERSION
fed-node        Ready       4h
```


* 删除 node：

为从 Kubernetes 集群中删除 _fed-node_ ，用户应当在 fed-master 上运行以下命令（请勿执行，这里只是为了提供信息）：

```shell
kubectl delete -f ./node.json
```


*您应该完成了!*


**cluster 应该在运行了! 启动一个测试 pod。**


## 支持级别


IaaS 提供商           | 配置管理      |操作系统 | 网络        | 文档                                               | 遵从     | 支持级别
-------------------- | ------------ | ------ | ----------  | ---------------------------------------------     | ---------| ----------------------------
裸金属                | 自定义        | Fedora | _无_        | [docs](/docs/getting-started-guides/fedora/fedora_manual_config)            |          | 项目


想要了解所有解决方案相关的支持级别信息，请查看 [解决方案表](/docs/getting-started-guides/#table-of-solutions)。

