---
approvers:
- dchen1107
- erictune
- thockin
cn-approvers:
- lichuqiang
title: Fedora（多节点）
---


* TOC
{:toc}


本文档介绍了如何在多台主机上部署 Kubernetes，来创建一个多节点集群，并使用 Flannel 组网。 请按照 fedora
[入门指南](/docs/getting-started-guides/fedora/fedora_manual_config/) 来设置 1 个 master
(fed-master) 以及 2 个或更多的 node。 确保所有的 node 有不同的名称（fed-node1、 fed-node2 等等）以及标签（fed-node1-label、 fed-node2-label 等等）来避免冲突。您还需要确保 Kubernetes master 主机运行着 etcd、
kube-controller-manager、 kube-scheduler 和 kube-apiserver 服务，同时 node 主机运行着 docker、
kube-proxy 和 kubelet 服务。 之后可以在 Kubernetes node 上安装 flannel。 每个 node 上的 Flannel 配置了一个
docker 使用的 overlay 网络。 Flannel 在每个 node 上运行，以建立一个唯一的 C 类容器网络。


## 前提

您需要 2 台或更多安装了 Fedora 的机器。 这些机器可以是裸金属机或虚拟机。


## Master 设置

**在 Kubernetes master 上执行以下命令**

* 通过在您的 fed-master 当前目录下创建一个 `flannel-config.json` 文件来配置 flannel。
Flannel 提供了 overlay 网络后端选项中的 udp 和 vxlan 类型。 在本指南中，我们选择基于 kernel 的 vxlan 后端。
json 文件的内容为：

```json
{
    "Network": "18.16.0.0/16",
    "SubnetLen": 24,
    "Backend": {
        "Type": "vxlan",
        "VNI": 1
     }
}
```


**注意：** 选择的 IP 地址范围*不能*是公共 IP 地址范围的一部分。

将该配置加入到 fed-master 上的 etcd 服务器中。

```shell
etcdctl set /coreos.com/network/config < flannel-config.json
```


* 验证关键字存在于 fed-master 上的 etcd 服务器中。

```shell
etcdctl get /coreos.com/network/config
```


## Node 设置

**在所有 Kubernetes node 上执行以下命令**

安装 flannel 包

```shell
# dnf -y install flannel
```


编辑 flannel 配置文件 /etc/sysconfig/flanneld 为如下形式：


```shell
# Flanneld 配置选项

# etcd url 位置。 将该配置指向 etcd 运行的服务器
FLANNEL_ETCD="http://fed-master:2379"

# etcd 配置关键字。 该关键字用于 flannel 查询
# 地址范围分配
FLANNEL_ETCD_KEY="/coreos.com/network"

# 您希望传入的任何其他选项
FLANNEL_OPTIONS=""
```


**注意：** 默认情况下，flannel 使用网络接口作为默认路由。 如果您拥有多个网络接口，并希望使用其中一个网络接口，
而不是默认路由的那一个，您可以添加 "-iface=" 参数到 FLANNEL_OPTIONS 中。 想了解附加选项的信息，
可以在命令行中运行 `flanneld --help`。

启用 flannel 服务。

```shell
systemctl enable flanneld
```


如果 docker 未在运行，那么启动 flannel 服务就足够了，跳过下一步。

```shell
systemctl start flanneld
```


如果 docker 已经在运行，那么停止 docker，删除 docker 网桥（docker0），按照如下方式启动 flanneld 并重启 docker。
另外一种选择是重启系统（`systemctl reboot`）。

```shell
systemctl stop docker
ip link delete docker0
systemctl start flanneld
systemctl start docker
```



## **测试集群和 flannel 配置**

现在检查 node 上的网络接口。 注意现在有了一个 flannel.1 接口，且 docker0 和 flannel.1 接口的 ip
地址处于同一网络中。 您会注意到每个 Kubernetes node 上的 docker0 都被分配了一个处于上面配置的
IP 地址范围之外的子网（如下面展示的 18.16.29.0/24）。 正常的输出应如下所示：

```shell
# ip -4 a|grep inet
    inet 127.0.0.1/8 scope host lo
    inet 192.168.122.77/24 brd 192.168.122.255 scope global dynamic eth0
    inet 18.16.29.0/16 scope global flannel.1
    inet 18.16.29.1/24 scope global docker0
```


从集群中的任一 node 上，通过 curl（使用 `grep -E "\{|\}|key|value"` 来输出部分结果）向 etcd 服务器发起查询，
来检查集群成员。 如果您设置了一个包含 1 个 master 和 3 个 node 的集群，您应当看到每个 node 对应一个信息块，
展示了它们被分配的子网。 您可以通过输出结果中列出的 MAC 地址（VtepMAC）和 IP 地址（公共 IP）将这些子网与每个
node 联系到一起。

```shell
curl -s http://fed-master:2379/v2/keys/coreos.com/network/subnets | python -mjson.tool
```

```json
{
    "node": {
        "key": "/coreos.com/network/subnets",
            {
                "key": "/coreos.com/network/subnets/18.16.29.0-24",
                "value": "{\"PublicIP\":\"192.168.122.77\",\"BackendType\":\"vxlan\",\"BackendData\":{\"VtepMAC\":\"46:f1:d0:18:d0:65\"}}"
            },
            {
                "key": "/coreos.com/network/subnets/18.16.83.0-24",
                "value": "{\"PublicIP\":\"192.168.122.36\",\"BackendType\":\"vxlan\",\"BackendData\":{\"VtepMAC\":\"ca:38:78:fc:72:29\"}}"
            },
            {
                "key": "/coreos.com/network/subnets/18.16.90.0-24",
                "value": "{\"PublicIP\":\"192.168.122.127\",\"BackendType\":\"vxlan\",\"BackendData\":{\"VtepMAC\":\"92:e2:80:ba:2d:4d\"}}"
            }
    }
}
```


在所有 node 上，检验 `/run/flannel/subnet.env` 文件。 该文件是由 flannel 自动生成的。

```shell
# cat /run/flannel/subnet.env
FLANNEL_SUBNET=18.16.29.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=false
```


这个时候，我们已经使得 etcd 运行在 Kubernetes master上，同时 flannel/docker 运行在 Kubernetes node 上。
下面的步骤是为了测试跨主机容器通信，这将证实 docker 和 flannel 是否配置正确。

```shell
# docker run -it fedora:latest bash
bash-4.3# 
```


上面的操作会使您处于容器中。 安装 iproute 和 iputils 包来安装 ip 和 ping 工具。 由于一个
[bug](https://bugzilla.redhat.com/show_bug.cgi?id=1142311)，您需要修改 ping 的二进制文件的权限来解决
"Operation not permitted" 错误。

```shell
bash-4.3# dnf -y install iproute iputils
bash-4.3# setcap cap_net_raw-ep /usr/bin/ping
```


现在注意第一个 node 上的 IP 地址：

```shell
bash-4.3# ip -4 a l eth0 | grep inet
    inet 18.16.29.4/24 scope global eth0
```


还要注意另一个 node 上的 IP 地址：

```shell
bash-4.3# ip a l eth0 | grep inet
    inet 18.16.90.4/24 scope global eth0
```

现在从第一个 node 来 ping 另一个 node：

```shell
bash-4.3# ping 18.16.90.4
PING 18.16.90.4 (18.16.90.4) 56(84) bytes of data.
64 bytes from 18.16.90.4: icmp_seq=1 ttl=62 time=0.275 ms
64 bytes from 18.16.90.4: icmp_seq=2 ttl=62 time=0.372 ms
```


现在使用由 flannel 创建的 overlay 网络的 Kubernetes 多节点集群已经创建完成了。


## 支持级别



IaaS 提供商           | 配置管理      | 操作系统| 网络        | 文档                                               | 遵从     | 支持级别
-------------------- | ------------ | ------ | ----------  | ---------------------------------------------     | ---------| ----------------------------
裸金属                | 自定义        | Fedora | flannel     | [文档](/docs/getting-started-guides/fedora/flannel_multi_node_cluster/)      |          | 社区 ([@aveshagarwal](https://github.com/aveshagarwal))
libvirt              | 自定义        | Fedora | flannel     | [文档](/docs/getting-started-guides/fedora/flannel_multi_node_cluster/)      |          | 社区 ([@aveshagarwal](https://github.com/aveshagarwal))
KVM                  | 自定义        | Fedora | flannel     | [文档](/docs/getting-started-guides/fedora/flannel_multi_node_cluster/)      |          | 社区 ([@aveshagarwal](https://github.com/aveshagarwal))
