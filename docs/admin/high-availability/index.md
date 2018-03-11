---
cn-approvers:
- xiaosuiba
- jimmysong
title: 构建高可用集群
---



## 简介

本文介绍了如何构建一个高可用（high-availability, HA）的 Kubernetes 集群。这是一个相当高级的话题。我们鼓励只想尝试性使用 Kubernetes 的用户使用更简单的配置，例如 [Minikube](/docs/getting-started-guides/minikube/)，或者尝试 [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) 提供的托管 Kubernetes 集群。

此外，目前我们并没有在端到端（e2e）测试中对 Kubernetes 的高可用性支持进行连续测试。我们正在努力添加这个持续测试项，但现在对单节点 master 安装方式测试得更加严格。

* TOC
{:toc}


## 概述

建立一个真正可靠的、高可用的分布式系统需要多个步骤。它类似于穿着内裤、裤子、腰带、吊带、另一条内裤和另一条裤子。我们将对每个步骤进行详细介绍，但是这里给出一个总结，用于帮助指导和引导用户。

涉及的步骤如下：

   * [创建可靠的组成节点，共同构成高可用 master 实现](#可靠的节点)
   * [建立一个冗余的、使用可靠存储层的 etcd 集群。](#建立一个冗余的，可靠的数据存储层)
   * [设置主选举（master-elected）的 Kubernetes scheduler 和 controller-manager 守护进程](#主选举组件)

这是系统完成时应该看起来的样子：

![高可用 Kubernetes 示意图](/images/docs/ha.svg)


## 初始设置

本指南的其余部分假设您正在设置一个 3 节点的集群 master，其中的每台机器上都运行着某种 Linux。本指南中的例子针对 Debian 发行版，但他们应该很容易的用于其他发行版上。
同样的，无论是在公有/私有云服务提供商上，还是在裸金属上运行集群，这些设置都应该可以工作。

实现高可用 Kubernetes 集群最简单的方法是从一个现有的单 master 集群开始。[https://get.k8s.io](https://get.k8s.io) 处的说明描述了在各种平台上安装单 master 集群的简单方法。


## 可靠的节点

在每个 master 节点上，我们都将运行一些实现 Kubernetes API 的进程。为了使这些进程可靠，第一步是保证在它们故障之后能够自动重启。为了实现这一点，我们需要安装一个进程监视器。我们选择使用每个工作节点上都会运行的 `kubelet`。这很方便，因为我们可以使用容器来分发我们的二进制文件、建立资源限额并审视每个守护进程的资源使用情况。当然，我们也需要一些机制来监控 kubelet 本身（在此插入一个"谁在监视监视者"的笑话）。对于 Debian 系统，我们选择 monit，但也存在一些替代的选择。例如，在基于 systemd 的系统（例如 RHEL，CentOS）上，您可以运行 'systemctl enable kubelet'。


如果您是从标准 Kubernetes 安装扩展而来，那么 `kubelet` 二进制文件应该已经存在于您的系统之中。您可以运行 `which kubelet` 来确定此文件是否已经安装。如果没有，您应该安装 [kubelet 二进制文件](https://storage.googleapis.com/kubernetes-release/release/v0.19.3/bin/linux/amd64/kubelet)、 [kubelet 初始化文件](http://releases.k8s.io/{{page.githubbranch}}/cluster/saltbase/salt/kubelet/initd) 和 [default-kubelet](/docs/admin/high-availability/default-kubelet) 脚本。

如果您正在使用 monit，您还应该安装 monit 守护进程（`apt-get install monit`）和 [monit-kubelet](/docs/admin/high-availability/monit-kubelet) 以及 [monit-docker](/docs/admin/high-availability/monit-docker) 配置。

在 systemd 系统中，您应该执行 `systemctl enable kubelet` 和 `systemctl enable docker`。


## 建立一个冗余的，可靠的数据存储层

高可用性解决方案的核心基础是冗余，可靠的存储层。高可用性的头号规则是保护数据。无论发生什么事情，如果你有数据，你还可以重建。如果失去了数据，你就完了。

集群化的 etcd 已将您的存储复制到了集群中的所有 master 实例上。这意味着要丢失数据，需要让全部三个节点的物理（或虚拟）磁盘同时失效。发生这种情况的可能性相对较低。所以对很多人来说，运行复制的 etcd 集群可能已经足够可靠了。如果仍然不够，您还可以添加 [更多的冗余存储层](#更可靠的存储)


### 集群化的 etcd

建立 etcd 集群的完整细节超出了本文的范围，[etcd 集群页面](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/clustering.md) 给出了大量的细节。本示例演练了一个简单集群的设置，使用 etcd 内置的发现机制来构建我们的集群。

首先，访问 etcd discovery 服务来创建一个新的令牌：

```shell
curl https://discovery.etcd.io/new?size=3
```


在每个节点上，将 [etcd.yaml](/docs/admin/high-availability/etcd.yaml) 文件复制到 `/etc/kubernetes/manifests/etcd.yaml` 中

每个节点上的 kubelet 会主动监视该目录的内容，并会按照 `etcd.yaml` 中对 pod 的定义创建一个 `etcd` 服务实例。

请注意，您应该将所有机中上 `etcd.yaml` 中的 `${DISCOVERY_TOKEN}` 替换为上面获得的令牌 URL，并将 `${NODE_NAME}` 替换为一个不同的名称（例如 `node-1`），以及将 `${NODE_IP}` 替换为每个机器的正确 IP 地址。 


#### 验证您的集群

一旦将其复制到所有三个节点中，您应该建立起了一个集群化的 etcd。您可以在 master 上这样验证：

```shell
kubectl exec < pod_name > etcdctl member list
```


以及：

```shell
kubectl exec < pod_name > etcdctl cluster-health
```


您还可以通过在一个节点上运行 `etcdctl set foo bar` 并在另一个节点上运行 `etcdctl get foo` 来验证其是否工作正常。


### 更可靠的存储

当然，如果您对提高数据可靠性感兴趣，还有其他选项可以让您找到比在普通磁盘上安装 etcd 数据更可靠的位置（皮带*和*吊带，ftw！）。

如果您使用云服务提供商，那么他们通常会提供此服务，例如 Google Cloud Platform 上的 [Persistent Disk](https://cloud.google.com/compute/docs/disks/persistent-disks)。这是一种可以挂载到虚拟机上的块设备持久性存储。其他云服务提供商也提供了类似的解决方案。


如果您在物理机器上运行，则可以使用通过网络连接的 iSCSI 或 NFS 接口的冗余存储。或者，您可以运行 Gluster 或 Ceph 等集群文件系统。最后，您还可以在每台物理机器上运行 RAID 阵列。

无论您选择如何实现，如果您选择使用这些选项之一，则应确保您的存储已挂载到每台机器上。如果您的存储在集群中的三个 master 之间共享，则应该在存储上为每个节点创建一个不同的目录。在所有的介绍中，我们都将假设这个存储被挂载到您机器中的 `/var/etcd/data` 目录。


## 复制的 API Server

一旦正确的设置好了复制的 etcd 集群，我们将使用 kubelet 安装 apiserver。

### 安装配置文件

首先，您需要创建初始日志文件，以便 Docker 挂载一个文件而不是目录：

```shell
touch /var/log/kube-apiserver.log
```


接下来，您在每个节点上创建一个 `/srv/kubernetes/` 目录。这个目录包含：

   * basic_auth.csv  - basic auth 用户名和密码
   * ca.crt - 证书颁发机构证书（Certificate Authority cert）
   * known_tokens.csv - 实体（例如 kubelet） 用于同 apiserver 通信的令牌
   * kubecfg.crt - 客户端证书，公钥
   * kubecfg.key - 客户端证书，私钥
   * server.cert - 服务器证书，公钥
   * server.key - 服务器证书，私钥

创建此目录的最简单方法可能是从一个工作集群的 master 节点复制它们，或者您可以自己手动生成这些文件。


### 启动 API Server

一旦创建了这些文件，请将 [kube-apiserver.yaml](/docs/admin/high-availability/kube-apiserver.yaml) 复制到每个 master 节点的 `/etc/kubernetes/manifests/` 目录下。

kubelet 监视这个目录，并且会使用文件中指定的 pod 定义自动创建一个 `kube-apiserver` 的容器实例。


### 负载均衡

此时，您应该有 3 个全部正常工作的 apiserver。如果您设置了一个网络负载均衡器，你就可以通过它访问您的集群，并平衡各个 apiserver 实例的流量。负载均衡器的配置取决于您的平台的具体情况，例如，Google Cloud Platform 的相关说明可以在 [这里](https://cloud.google.com/compute/docs/load-balancing/) 找到。


请注意，如果您启用了身份验证，可能需要重新生成证书，在每个独立节点的 IP 地址外，还应包含负载均衡器的IP地址。

对于部署到集群中的 pod 来说，`kubernetes` service/dns 名称自动的提供了一个 master 的负载均衡 endpoint。

对于 API 的外部用户（例如 `kubectl` 命令行接口、持续构建管道或其它客户端），您应该配置它们使用外部负载均衡器 IP 地址同 API 进行通信。


### Endpoint reconciler

如前一节所述，apiserver 通过一个名为 `kubernetes` 的 service 进行公开。这个 service 的 endpoint 对应于我们刚刚部署的 apiserver 集群。

由于更新 endpoint 和 service 需要 apiserver 启动，apiserver 中有特殊的代码可以使其直接更新自己的 endpoint。这个代码被称为“reconciler（协调器）”，因为它会对存储在 etcd 中及正在运行的 endpoint 列表进行协调。


在 Kubernetes 1.9 版本之前，reconciler 希望您通过一个命令行参数（例如 `--apiserver-count=3`）提供 endpoint 的数量（意即 apiserver 的副本数）。如果有更多可用的副本，reconciler 将对 endpoint 列表进行截取。因此，如果一个运行 apiserver 副本的节点宕机并被替换，endpoint 列表将最终被更新。然而，在副本被替换前，它的 endpoint 都将留存在列表中。在此期间，一小部分发送到 `kubernetes` service 的 API 请求将会失败，因为它们会被发送到一个未运行的 endpoint。

这就是上一节建议您部署负载均衡器并通过它访问 API 的原因。这个 负载均衡器将直接评估 apiserver 的健康状态，确保请求不会发送到崩溃的示例。


如果您不添加 `--apiserver-count` 参数，其值将默认为 1。您的集群将会正常工作，但每个 apiserver 副本都会持续尝试在删除其它 endpoint 时将自己添加到列表中，这将在 kube-proxy 和其他组件中产生大量无用的更新。

从  Kubernetes 1.9 版本开始，有了一个新的 reconciler 实现。它使用了一个被每个 apiserver 副本定期更新的*租约*。当副本宕机时，它会停止更新自己的租约，其他副本注意到这个租约过期并从 endpoint 列表中将其删除。您可以在启动 apiserver 副本时，通过添加 `--endpoint-reconciler-type=lease` 参数来切换到新的 reconciler。


如果您希望了解更多信息，可以查看以下资源：
- [issue kubernetes/kuberenetes#22609](https://github.com/kubernetes/kubernetes/issues/22609)，给出了更多的上下文。
- [master/reconcilers/mastercount.go](https://github.com/kubernetes/kubernetes/blob/dd9981d038012c120525c9e6df98b3beb3ef19e1/pkg/master/reconcilers/mastercount.go#L63)，基于 master 计数的 reconciler 实现。
- [PR kubernetes/kubernetes#51698](https://github.com/kubernetes/kubernetes/pull/51698)，添加了对基于租约的 reconciler 的支持。


## 主选举的组件

到目前为止，我们已经建立起了状态存储，并且已经建立了 API server，但我们还没有运行任何实际修改集群状态的内容，例如 controller manager 和 scheduler。为了可靠的实现这一点，我们每次只希望一个 actor 修改状态，但是我们希望复制这些 actor 实例，以防止有机器宕机。为了达到这个目的，我们将在 API 中使用一个租约锁来执行主选举（master election）。我们将对每个 scheduler 和 controller manager 使用 `--leader-elect` 参数，以在 API 中使用租约来确保同一时间只有一个 scheduler 和 controller-manager 实例运行。


scheduler 和 controller-manager 可以配置为和相同节点（意即 127.0.0.1）上的 API server 进行通信，或者也可以配置为使用 API server 的负载均衡器地址。不管如何配置它们，当使用 `--leader-elect` 参数时，scheduler 和 controller-manager 都将完成上文提到的主选举过程。

当无法访问 API server 时，选举的 leader 无法更新其租约，这将导致选举产生新的 leader。在 scheduler 和 controller-manager 通过 127.0.0.1 访问 API server 并且相同节点上的 API server 宕机时，这一点显得尤为重要。


### 安装配置文件

首先，在每个节点上创建空的日志文件，这样 Docker 将挂载一个文件而不是创建新目录：

```shell
touch /var/log/kube-scheduler.log
touch /var/log/kube-controller-manager.log
```


接下来，通过复制  [kube-scheduler.yaml](/docs/admin/high-availability/kube-scheduler.yaml) 和 [kube-controller-manager.yaml](/docs/admin/high-availability/kube-controller-manager.yaml) 到 `/etc/kubernetes/manifests/` 目录来建立每个节点的 scheduler 和 controller manager 的 pod 的配置描述文件。


## 结论

目前，您已经完成了 master 组件的配置（耶！），但您仍然需要添加工作节点（噗！）。

如果您有一个存在的集群，这就很简单，只需要重新配置 kubelet 与负载均衡器的 endpoint 通信，并重启每个节点上的 kubelet。

如果您建立的是一个新集群，您将需要在每个工作节点上安装 kubelet 和 kube-proxy，并且将 `--apiserver` 参数设置为您的复制的 endpoint。
