---
cn-approvers:
- linyouchong
approvers:
- davidopp
title: 使用 Salt 配置 Kubernetes 集群
---



Kubernetes 集群可以使用 Salt 进行配置


这些 Salt 脚本可以跨多个托管提供商共享，这取决于您在何处托管 Kubernetes 集群，您可能正在使用多种不同的操作系统和多种不同的网络配置。因此，在做修改 Salt 配置之前了解一些背景信息是很重要的，以便在使用其他主机托管提供商时降低集群配置失败的可能。


## 创建 Salt 集群


**salt-master** 服务运行在 kubernetes-master 节点 [（除了在默认的 GCE 环境和 OpenStack-Heat 环境）](#standalone-salt-configuration-on-gce-and-others)。


**salt-minion** 服务运行在 kubernetes-master 节点和每个 kubernetes-node 节点。


每个 salt-minion 服务在 **master.conf** 文件中配置与 kubernetes-master 节点上的 **salt-master** 服务进行交互 [（除了 GCE 环境和 OpenStack-Heat 环境）](#standalone-salt-configuration-on-gce-and-others)。

```shell
[root@kubernetes-master] $ cat /etc/salt/minion.d/master.conf
master: kubernetes-master
```


每个 salt-minion 都会与 salt-master 联系，根据其提供的机器信息，salt-master 会向其提供作为 kubernetes-master 或 kubernetes-node 用于运行 Kubernetes 所需要的能力。


如果您正使用基于 Vagrant 的环境，**salt-api** 服务运行在 kubernetes-master 节点。它被配置为使 Vagrant 用户能够对 Salt 集群进行内省，以便通过 REST API 了解 Vagrant 环境中的机器的信息。


## 在 GCE 和其它环境下独立配置 Salt


在 GCE 和 OpenStack 环境，使用 Openstack-Heat 提供商，master 和 node 节点被配置为 [standalone minions](http://docs.saltstack.com/en/latest/topics/tutorials/standalone_minion.html)。每个 VM 的配置都源于它的 [instance metadata](https://cloud.google.com/compute/docs/metadata) 并被保存在 Salt grains (`/etc/salt/minion.d/grains.conf`) 和本地 Salt 用于保存执行状态的 pillars (`/srv/salt-overlay/pillar/cluster-params.sls`) 中。


对于 GCE 和 OpenStack ，所有引用 master/minion 设置的其余部分都应该被忽略。这种设置的一个后果是，Salt 不存在 - 节点之间不存在配置共享。


## Salt 安全


*（不适用于 默认的 GCE 和 OpenStack-Heat 配置环境。）*


salt-master 没有启用安全功能，salt-master 被配置为自动接受所有来自 minion 的接入请求。在深入研究之前，不推荐在生产环境中启用安全配置。（在某些环境中，如果 salt-master 端口不能从外部访问，并且您信任您的网络上的每个节点，这并不像它看起来那么糟糕）

```shell
[root@kubernetes-master] $ cat /etc/salt/master.d/auto-accept.conf
open_mode: True
auto_accept: True
```


## 配置 Salt minion


Salt 集群中的每个 minion 都有一个相关的配置，它指示 salt-master 如何在机器上提供所需的资源。


下面是一个基于 Vagrant 环境的示例文件：

```shell
[root@kubernetes-master] $ cat /etc/salt/minion.d/grains.conf
grains:
  etcd_servers: $MASTER_IP
  cloud: vagrant
  roles:
    - kubernetes-master
```


每个托管环境都使用了略微不同的 grains.conf 文件，用于在需要的 Salt 文件中构建条件逻辑。


下面列举了目前支持的定义键/值对的集合。如果你添加了新的，请确保更新这个列表。


键 | 值
-----------------------------------|----------------------------------------------------------------

`api_servers` | （可选） IP 地址/主机名 ，kubelet 用其访问 kube-apiserver

`cbr-cidr` | （可选） docker 容器网桥分配给 minion 节点的 IP 地址范围

`cloud` | （可选） 托管 Kubernetes 的 IaaS 平台， *gce*, *azure*, *aws*, *vagrant*

`etcd_servers` | （可选） 以逗号分隔的 IP 地址列表，kube-apiserver 和 kubelet 使用其访问 etcd。kubernetes_master 角色的节点使用第一个机器的 IP ，在 GCE 环境上使用 127.0.0.1。

`hostnamef` | （可选） 机器的完整主机名，即：uname -n

`node_ip` | （可选）用于定位本节点的 IP 地址

`hostname_override` | （可选）对应 kubelet 的 hostname-override 参数

`network_mode` | （可选）节点间使用的网络模型：*openvswitch*

`networkInterfaceName` | （可选）用于绑定地址的网络接口，默认值 *eth0*

`publicAddressOverride` | （可选）kube-apiserver 用于绑定外部只读访问的IP地址

`roles` | （必选）1、`kubernetes-master` 表示本节点是 Kubernetes 集群的 master。2、`kubernetes-pool` 表示本节点是一个 kubernetes-node。根据角色，Salt 脚本会在机器上提供不同的资源


Salt sls 文件可以应用这些键到分支行为。


此外，一个集群可能运行在基于 Debian 的操作系统或基于 Red Hat 的操作系统（Centos、Fedora、RHEL等）。因此，有时区分基于操作系统的行为（如果像下面这样的分支）是很重要的。

```liquid
{% raw %}
{% if grains['os_family'] == 'RedHat' %}
// something specific to a RedHat environment (Centos, Fedora, RHEL) where you may use yum, systemd, etc.
{% else %}
// something specific to Debian environment (apt-get, initd)
{% endif %}
{% endraw %}
```


## 最佳实践


在为进程配置默认参数时，最好避免使用环境文件（ Red Hat 环境中的 Systemd ）或 init.d 文件（ Debian 发行版）以保留在操作系统环境中应该通用的默认值。这有助于保持我们的 Salt 模板文件易于理解，因为管理员可能不熟悉每个发行版的细节。


## 未来的增强（网络）


每个 pod IP 配置都是特定于提供商的，因此在进行网络更改时，必须将这些设置为沙箱，因为不同的提供商可能不会使用相同的机制（ iptables 、openvswitch 等）。


我们应该定义一个 grains.conf 键，这样能更明确地捕获正在使用的网络环境配置，以避免将来在不同的提供商之间产生混淆。


## 进一步阅读


[cluster/saltbase](http://releases.k8s.io/{{page.githubbranch}}/cluster/saltbase/) 项目有更多关于当前 SaltStack 配置的详细信息。
