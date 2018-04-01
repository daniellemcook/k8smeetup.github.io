---
approvers:
- mml
- wojtek-t
title: 为 Kubernetes 运行 etcd 集群
cn-approvers:
- chentao1596
---


## 先决条件


* 运行的 etcd 集群有奇数个成员。


* etcd 是一个 leader-based 分布式系统。确保领导定期向所有追随者发送心跳，以保持集群稳定。


* 确保不发生资源饥饿。


* 集群的性能和稳定性对网络和磁盘 IO 非常敏感。任何资源饥饿都会导致心跳超时，从而导致集群的不稳定。不稳定的情况表明没有选出任何领导人。在这种情况下，集群不能对其当前状态进行任何更改，这意味着不能调度新的 pod。


* 保持稳定的 etcd 集群对 Kubernetes 集群的稳定性至关重要。因此，请在专用机器或隔离环境上运行 etcd 集群，以满足 [所需资源保证](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/hardware.md#hardware-recommendations)。


## 资源要求


使用有限的资源运行 etcd 只适合测试目的。为了在生产中部署，需要先进的硬件配置。在生产中部署 etcd 之前，请查看 [所需资源参考文档](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/hardware.md#example-hardware-configurations)。


## 启动 Kubernetes API 服务器


本节介绍如何在部署中使用 etcd 集群启动 Kubernetes API 服务器。


### 单节点 etcd 集群


只为测试目的使用单节点 etcd 集群。


1. 运行以下命令：

        ./etcd --client-listen-urls=http://$PRIVATE_IP:2379 --client-advertise-urls=http://$PRIVATE_IP:2379


2. 使用标志 `--etcd-servers=$PRIVATE_IP:2379` 启动 Kubernetes API 服务器。
   
   使用您 etcd 客户端 IP 替换 `PRIVATE_IP`。


### 多节点 etcd 集群


为了耐用性和高可用性，在生产中将以多节点集群的方式运行 etcd，并且定期备份。建议在生产中使用五个成员的集群。有关该内容的更多信息，请参阅 [常见问题文档](https://github.com/coreos/etcd/blob/master/Documentation/faq.md#what-is-failure-tolerance)。


可以通过静态成员信息或动态发现的方式配置 etcd 集群。有关集群的详细信息，请参阅 [etcd 集群文档](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/clustering.md)。


例如，考虑运行以下客户端 URL 的五个成员的 etcd 集群：`http://$IP1:2379`，`http://$IP2:2379`，`http://$IP3:2379`，`http://$IP4:2379` 和 `http://$IP5:2379`。要启动 Kubernetes API 服务器：


1. 运行以下命令：

       ./etcd --client-listen-urls=http://$IP1:2379, http://$IP2:2379, http://$IP3:2379, http://$IP4:2379, http://$IP5:2379 --client-advertise-urls=http://$IP1:2379, http://$IP2:2379, http://$IP3:2379, http://$IP4:2379, http://$IP5:2379


2. 使用标志 `--etcd-servers=$IP1:2379, $IP2:2379, $IP3:2379, $IP4:2379, $IP5:2379` 启动 Kubernetes API 服务器。

    使用您 etcd 客户端 IP 替换 `PRIVATE_IP`。


### 使用负载均衡的多节点 etcd 集群


要运行负载均衡的 etcd 集群：


1. 建立一个 etcd 集群。

2. 在 etcd 集群前面配置负载均衡器。例如，让负载均衡器的地址为 `$LB`。

3. 使用标志 `--etcd-servers=$LB:2379` 启动 Kubernetes API 服务器。


## 安全的 etcd 集群


对 etcd 的访问相当于集群中的 root 权限，因此理想情况下只有 API 服务器才能访问它。考虑到数据的敏感性，建议只向需要访问 etcd 集群的节点授予权限。


想要确保 etcd 的安全，可以设置防火墙规则或使用 etcd 提供的安全特性，这些安全特性依赖于 x509 公钥基础设施（PKI）。首先，通过生成密钥和证书对来建立安全的通信通道。例如，使用密钥对 `peer.key` 和 `peer.cert` 来保护 etcd 成员之间的通信，而 `client.cert` 和 `client.cert` 用于保护 etcd 与其客户端之间的通信。请参阅 etcd 项目提供的 [示例脚本](https://github.com/coreos/etcd/tree/master/hack/tls-setup)，以生成用于客户端身份验证的密钥对和 CA 文件。


### 安全通信


若要使用安全对等通信对 etcd 进行配置，请指定标志 `--peer-key-file=peer.key` 和 `--peer-cert-file=peer.cert`，并使用 https 作为 URL 模式。


类似地，要使用安全客户端通信对 etcd 进行配置，请指定标志 `--key-file=k8sclient.key` 和 `--cert-file=k8sclient.cert`，并使用 https 作为 URL 模式。


### 限制 etcd 集群的访问


配置安全通信后，将 etcd 集群的访问限制在 Kubernetes API 服务器上。使用 TLS 身份验证来完成此任务。


例如，考虑由 CA `etcd.ca` 信任的密钥对 `k8sclient.key` 和 `k8sclient.cert`。当 etcd 配置为 `--client-cert-auth` 和 TLS 时，它使用系统 CA 或由 `--trusted-ca-file` 标志传入的 CA 验证来自客户端的证书。指定标志 `--client-cert-auth=true` 和 `--trusted-ca-file=etcd.ca` 将限制对具有证书 `k8sclient.cert` 的客户端的访问。


一旦正确配置了 etcd，只有具有有效证书的客户端才能访问它。要让 Kubernetes API 服务器访问，可以使用标志 `--etcd-certfile=k8sclient.cert` 和 `--etcd-keyfile=k8sclient.key` 配置它。


**注意**：Kubernetes 目前不支持 etcd 身份验证。想要了解更多信息，请参阅相关的问题 [支持 etcd v2 的基本认证](https://github.com/kubernetes/kubernetes/issues/23398)。 
{: .note}


## 替换失败的 etcd 成员


etcd 集群通过容忍少数成员故障实现高可用性。但是，要改善集群的整体健康状况，请立即替换失败的成员。当多个成员失败时，逐个替换它们。替换失败成员需要两个步骤：删除失败成员和添加新成员。


虽然 etcd 在内部保留唯一的成员 ID，但建议为每个成员使用唯一的名称，以避免人为错误。例如，考虑一个三成员的 etcd 集群。让 URL 为：member1=http://10.0.0.1， member2=http://10.0.0.2 和 member3=http://10.0.0.3。当 member1 失败时，将其替换为 member4=http://10.0.0.4。


1. 获取失败的 member1 的成员 ID：

    `etcdctl --endpoints=http://10.0.0.2,http://10.0.0.3 member list`
	
      显示以下信息：

        8211f1d0f64f3269, started, member1, http://10.0.0.1:12380, http://10.0.0.1:2379
        91bc3c398fb3c146, started, member2, http://10.0.0.1:2380, http://10.0.0.2:2379
        fd422379fda50e48, started, member3, http://10.0.0.1:2380, http://10.0.0.3:2379


2. 移除失败的成员

    `etcdctl member remove 8211f1d0f64f3269`
	
      显示以下信息：

       Removed member 8211f1d0f64f3269 from cluster


3. 增加新成员

    `./etcdctl member add member4 --peer-urls=http://10.0.0.4:2380`

      显示以下信息：

       Member 2be1eb8f84b7f63e added to cluster ef37ad9dc622a7c4


4. 在 IP 为 `10.0.0.4` 的机器上启动新增加的成员：

        export ETCD_NAME="member4"
        export ETCD_INITIAL_CLUSTER="member2=http://10.0.0.2:2380,member3=http://10.0.0.3:2380,member4=http://10.0.0.4:2380"
        export ETCD_INITIAL_CLUSTER_STATE=existing
        etcd [flags]


5. 做以下事情之一：

   1. 更新其 `--etcd-servers` 标志，使 Kubernetes 知道配置进行了更改，然后重新启动 Kubernetes API 服务器。
   2. 如果在部署中使用了负载均衡，更新负载均衡配置。


有关集群重新配置的详细信息，请参阅 [etcd 重构文档](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/runtime-configuration.md#remove-a-member)。


## 备份 etcd 集群


所有 Kubernetes 对象都存储在 etcd 上。定期备份 etcd 集群数据对于在灾难场景（例如丢失所有主节点）下恢复 Kubernetes 集群非常重要。快照文件包含所有 Kubernetes 状态和关键信息。为了保证敏感的 Kubernetes 数据的安全，可以对快照文件进行加密。


备份 etcd 集群可以通过两种方式完成：etcd 内置快照和卷快照。


### 内置快照


etcd 支持内置快照，因此备份 etcd 集群很容易。快照可以从使用 `etcdctl snapshot save` 命令的活动成员中获取，也可以通过从 etcd [数据目录](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/configuration.md#--data-dir) 复制 `member/snap/db` 文件，该 etcd 数据目录目前没有被 etcd 进程使用。`datadir` 位于 `$DATA_DIR/member/snap/db`。获取快照通常不会影响成员的性能。


下面是一个示例，用于获取 `$ENDPOINT` 所提供的键空间的快照到文件 `snapshotdb`：


```sh
ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT snapshot save snapshotdb
# exit 0

# 验证快照
ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshotdb
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| fe01cf57 |       10 |          7 | 2.1 MB     |
+----------+----------+------------+------------+
```


### 卷快照


如果 etcd 运行在支持备份的存储卷（如 Amazon 弹性块存储）上，则可以通过获取存储卷的快照来备份 etcd 数据。


## 扩大 etcd 集群


通过交换性能，扩展 etcd 集群可以提高可用性。缩放不会提高集群性能和能力。一般情况下不要扩大或缩小 etcd 集群的集合。不要为 etcd 集群配置任何自动缩放组。强烈建议始终在任何官方支持的规模上运行生产 Kubernetes 集群时使用静态的五成员 etcd 集群。


合理的扩展是在需要更高可靠性的情况下，将三成员集群升级为五成员集群。请参阅 [etcd 重新配置文档](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/runtime-configuration.md#remove-a-member) 以了解如何将成员添加到现有集群中的信息。


## 恢复 etcd 集群


etcd 支持从 [major.minor](http://semver.org/) 或其他不同 patch 版本的 etcd 进程中获取的快照进行恢复。还原操作用于恢复失败的集群的数据。


在启动还原操作之前，必须有一个快照文件。它可以是来自以前备份操作的快照文件，也可以是来自剩余 [数据目录](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/configuration.md#--data-dir) 的快照文件。`datadir` 位于 `$DATA_DIR/member/snap/db`。有关从快照文件还原集群的详细信息和示例，请参阅 [etcd 灾难恢复文档](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/recovery.md#restoring-a-cluster)。


如果还原的集群的访问 URL 与前一个集群不同，则必须相应地重新配置 Kubernetes API 服务器。在本例中，使用标志 `--etcd-servers=$NEW_ETCD_CLUSTER` 而不是标志 `--etcd-servers=$OLD_ETCD_CLUSTER` 重新启动 Kubernetes API 服务器。用相应的 IP 地址替换 `$NEW_ETCD_CLUSTER` 和 `$OLD_ETCD_CLUSTER`。如果在 etcd 集群前面使用负载平衡，则可能需要更新负载均衡器。


如果大多数 etcd 成员永久失败，则认为 etcd 集群失败。在这种情况下，Kubernetes 不能对其当前状态进行任何更改。虽然已调度的 pod 可能继续运行，但新的 pod 无法调度。在这种情况下，恢复 etcd 集群并可能需要重新配置 Kubernetes API 服务器以修复问题。


## 升级和回滚 etcd 集群


### 重要假设


本文件中描述的升级过程假定存在以下情况之一：

1. etcd 集群仅有一个节点。
2. etcd 集群有多个节点。

   在这种情况下，升级过程需要关闭 etcd 集群。在关闭 etcd 集群期间，Kubernetes API 服务器将只能读。


**警告**：对假设的偏差未经连续集成的测试，而且偏差可能会造成不良后果。有关操作 etcd 集群的其他信息可 [从 etcd 维护人员](https://github.com/coreos/etcd/tree/master/Documentation) 获得。
{: .warning}


###  背景


在 Kubernetes 版本 1.5.1 中，我们仍然使用 2.2.1 版本的 etcd 和 v2 API。此外，我们没有预先存在的进程来更新 etcd，因为我们从来没有更新过 etcd，无论是小版本还是主版本。


请注意，我们需要迁移我们正在使用的 etcd 版本（从 2.2.1 迁移到至少 3.0.x）以及 Kubernetes 与其通信的 etcd API 的版本。etcd 3.0.x 二进制文件同时支持 v2 和 v3 API。


本文档描述如何进行此迁移。如果您想跳过背景直接进入过程，请参阅 [升级过程](#升级过程)。


### etcd 升级要求


对于如何执行 etcd 集群升级有要求。主要考虑因素是：
- 一次升级一个小版本
- 通过附加工具支持回滚


#### 一次升级一个小版本


一次只升级一个小版本。例如，我们不能直接从 2.1.x 升级到 2.3.x。在补丁版本中，可以在任意版本之间进行升级和降级。为任何中间版本启动集群，等待集群正常运行，然后关闭集群将执行迁移。例如，要从 2.1.x 升级到 2.3.y，只需在 2.2.z 版本中启动 etcd，等待它正常运行，停止它，然后启动 2.3.y 版本。


#### 通过附加工具回滚


版本 3.0+ 的 etcd 不支持通用回滚。也就是说，在从 M.N 迁移到 M.N+1 之后，就没有办法返回 M.N 了。etcd 团队提供了一个 [自定义回滚工具](https://git.k8s.io/kubernetes/cluster/images/etcd/rollback)，但回滚工具有以下限制：


* 这个自定义的回滚工具不是 etcd 仓库的一部分，并且没有接受与 etcd 的其他部分相同的测试。我们正在几个端到端的测试中测试它。这里只有社区级别的支持。


* 回滚仅可以从 3.0.x 版本（即使用 v3 API）到 2.2.1 版本（即使用 v2 API）。


* 只有当数据以 `application/json` 格式存储时，该工具才能工作。


* 回滚不保留存储在 etcd 中的对象的资源版本。


**警告**：如果数据不以 `application/json` 格式保存（请参阅 [升级过程](#升级过程)），您将失去回滚到 etcd 2.2 的选项。
{: .warning}


最后一项意味着，任何组件或用户如果有一些逻辑依赖于资源版本，则可能需要在 etcd 回滚之后重新启动。这包括所有使用 watch API 的客户端，该 API 依赖于资源版本。由于 kubelet 和 Kube-Proxy 都使用 watch API，所以回滚可能需要重新启动所有 node 上的所有 Kubernetes 组件。


**注意**：在编写本文档时，Kubelet 和 KubeProxy 使用的 “resource version” 都只用于 watch（即资源版本不用于其它任何事情)。而且两者都在使用 reflector 和/或 informer 框架来 watch（也就是说，他们自己不发送 watch 请求）。如果这两个框架都不能更新 watch，那么它们将从 “current version” 开始，执行 “list + watch
from the resource version returned by list” 操作。这意味着，如果 apiserver 在回滚期间会关闭，那么所有节点组件基本上都应该重新启动它们的 watch，并在 apiserver 正常后从 “now” 开始。并且它将带着新的资源版本回来。这意味着不需要重新启动节点组件。但这里的假设可能永远不会成立。
{: .note}


### 设计


本节描述了我们如何去做迁移，给出 [etcd 升级要求](#etcd-升级要求)。


请注意，由于支持 etcd v3 API 所需的 Kubernetes 代码更改是只是在本地的和简单的，所以我们根本不关注它们。我们只关注这里的升级/回滚。


### 新的 etcd Docker 镜像


我们决定彻底改变 etcd 镜像的内容和它的工作方式。到目前为止，X 版中用于 etcd 的 Docker 镜像只包含 etcd 和 etcdctl 二进制文件。


接下来，X 版中用于 etcd 的 Docker 映像将包含多个版本的 etcd。例如，3.0.17 镜像将包含 etcd 和 etcdctl 的 2.2.1、2.3.7 和 3.0.17 二进制文件。这将允许使用相同的 Docker 镜像运行多个不同版本的 etcd。


此外，镜像将包含一个由 Kubernetes 团队编写的定制脚本，用于在版本之间进行迁移。该镜像还将包含 etcd 团队提供的回滚工具。


### 迁移脚本
迁移脚本将是 etcd Docker 镜像的一部分，它是一个 bash 脚本，其工作方式如下：


1. 检测我们之前运行的 etcd 版本。为此，我们添加了一个专用文件 `version.txt`，该文件保存了该信息并将它存储在 etcd-data-specific 目录中。如果该文件不存在，则默认为 2.2.1 版本。

1. 如果我们目前是 2.2.1 版本，并且想要升级，那么备份数据。

1. 基于检测到的前一个 etcd 版本和所需的版本（通过环境变量通信），按照需要执行升级步骤。这意味着，对于检测到的版本和期望的版本之间的每一个小版本：

   1. 启动该版本的 etcd。

   1. 等待直到它是健康的。健康意味着您可以给它写一些数据。

   1. 停止 etcd。请注意，该 etcd 不会侦听默认的 etcd 端口。要侦听 API 服务器未配置为连接的端口是很难的，这意味着 API 服务器将无法连接到它。假设没有其他客户端试图连接和写入这个隐形的端口，那么在此期间将不会写入任何新的数据。
   

1. 如果所需的 API 版本为 v3，而检测到的版本为 v2，则执行从 v2 到 v3 数据格式的离线迁移。为此，我们使用两个工具：
   * ./etcdctl migrate：这是 etcd 团队提供的官方迁移工具。
   * 将 TTLs 附加到 etcd 中的事件的自定义脚本。注意，etcdctl 迁移不支持 TTLs。

1. 每一步成功后，更新版本文件的内容。这将保护我们免受某些东西同时崩溃的时候导致版本文件与实际数据完全不同步的情形。请注意，如果脚本在步骤完成后和文件更新之前崩溃，则是安全的。这只会导致在下一次尝试中重做一步。


前面的所有步骤都适用于检测到的版本小于或等于所需版本的情况。在相反的情况下，即回滚，脚本的工作方式如下：


1. 使用 v3 API 验证检测到的版本为 3.0.x，使用 v2 API 验证所需的版本为 2.2.1。我们不支持任何其他回滚。

1. 如果是这样的话，我们将运行 etcd 团队提供的自定义工具来执行离线回滚。此工具读取 v3 格式的数据并以 v2 格式将其写回磁盘。

1. 最后更新版本文件的内容。


### 升级过程
只需将 etcd 清单（manifest）中的命令行修改为：


1. 运行迁移脚本。如果以前运行的版本已经在所需的版本中，该步骤无需操作。
1. 在所需的版本中启动 etcd。


从 Kubernetes 1.6 版本开始，这已经在新的谷歌计算引擎（GCE Google Compute Engine）集群的清单中完成了。您还应该指定下面这些环境变量。特别是，如果希望保留回滚的选项，则必须将 `STORAGE_MEDIA_TYPE` 设置为 `application/json`。

```
TARGET_STORAGE=etcd3
ETCD_IMAGE=3.0.17
TARGET_VERSION=3.0.17
STORAGE_MEDIA_TYPE=application/json
```


若要回滚，请使用以下内容：

```
TARGET_STORAGE=etcd2
ETCD_IMAGE=3.0.17
TARGET_VERSION=2.2.1
STORAGE_MEDIA_TYPE=application/json
```


## etcd 2.2.1 版本附注


### 默认配置


默认的安装脚本使用 kubelet 的基于文件的静态 pod 特性在 [pod](http://releases.k8s.io/{{page.githubbranch}}/cluster/gce/manifests/etcd.manifest) 中运行 etcd。此清单只应在 master 所在的 VM 上运行。kubelet 扫描清单的默认位置是 `/etc/kubernetes/manifests/`。


### Kubernetes 使用的 etcd


默认情况下，Kubernetes 对象存储在 etcd 的 `/registry` 键下。这个路径可以使用 [kube-apiserver](/docs/admin/kube-apiserver) 的标志 `--etcd-prefix="/foo"` 作为前缀。


`etcd` 是 Kubernetes 保持状态的唯一地方。


### 问题排查


若要测试 `etcd` 是否正确运行，可以尝试将值写入测试键。在您的 master 所在的 VM（或者某个配置了防火墙以便可以与集群的 etcd 进行通信的地方）上，尝试：

```shell
curl -X PUT "http://${host}:${port}/v2/keys/_test"
```
