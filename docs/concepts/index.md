---
cn-approvers:
- tianshapjq
title: 概念
---



本概念章节可帮助您了解 Kubernetes 系统的各个部分以及 Kubernetes 用于表示集群的抽象概念，并帮助您深入了解 Kubernetes 的工作原理。


## 概述


要使用 Kubernetes，您可以使用 *Kubernetes API 对象* 来描述您集群的 *所需状态*：您要运行的应用程序或其他工作负载、使用的容器镜像、副本数量和要使用的网络和磁盘资源 等等。您可以通过使用 Kubernetes API 创建对象（通常通过命令行界面 `kubectl` ）来设置所需的状态，也可以直接使用 Kubernetes API 与集群进行交互，并设置或修改您所需的状态。


一旦您设置了想要的状态，*Kubernetes 控制平面* 将会使集群的当前状态匹配所需的状态。为此，Kubernetes 自动执行各种任务--例如启动或重新启动容器、伸缩给定应用程序的副本数量等等。Kubernetes 控制平面由运行在集群上的一系列进程组成：


* ** Kubernetes Master ** 是由您集群中单个节点上运行的三个进程组成的，该节点被指定为 master 节点。这些进程包括：[kube-apiserver](/docs/admin/kube-apiserver/)、[kube-controller-manager](/docs/admin/kube-controller-manager/) 和 [kube-scheduler](/docs/admin/kube-scheduler/)。
* 每个集群的非 master 节点运行两个进程：
  * **[kubelet](/docs/admin/kubelet/)**，它和 Kubernetes Master 进行通信。
  * **[kube-proxy](/docs/admin/kube-proxy/)**，在每个节点上反映 Kubernetes 网络服务的网络代理。


## Kubernetes 对象


Kubernetes 包含许多代表系统状态的抽象概念：已部署的容器化应用程序和工作负载，与它们相关的网络和磁盘资源，以及有关您的集群所做事情的其他信息。这些抽象概念是由 Kubernetes API 中的对象来表示的; 请参阅 [Kubernetes 对象概述](/docs/concepts/abstractions/overview/) 以了解更多详情。


Kubernetes 的基本对象包括：

* [Pod](/docs/concepts/workloads/pods/pod-overview/)
* [Service](/docs/concepts/services-networking/service/)
* [Volume](/docs/concepts/storage/volumes/)
* [Namespace](/docs/concepts/overview/working-with-objects/namespaces/)


另外，Kubernetes 还包含一些称为控制器的更高层次的抽象概念。控制器基于基本对象，并提供附加特性和便利特性。包括：

* [ReplicaSet](/docs/concepts/workloads/controllers/replicaset/)
* [Deployment](/docs/concepts/workloads/controllers/deployment/)
* [StatefulSet](/docs/concepts/workloads/controllers/statefulset/)
* [DaemonSet](/docs/concepts/workloads/controllers/daemonset/)
* [Job](/docs/concepts/workloads/controllers/jobs-run-to-completion/)


## Kubernetes 控制平面


Kubernetes 控制平面的各个部分（如 Kubernetes Master 和 Kubelet 进程）管理 Kubernetes 如何与您的集群进行通信。控制平面保存系统中所有 Kubernetes 对象的记录，并运行连续的控制循环来管理这些对象的状态。在任何给定时间，控制平面的控制回路将响应集群中的变化，并使系统中所有对象的实际状态与您提供的所需状态相匹配。


例如，当您使用 Kubernetes API 创建一个 Deployment 对象时，您就为系统提供了一个新的所需状态。Kubernetes 控制平面记录了对象的创建，并通过启动所需的应用程序并将它们调度到集群节点来执行您的指令--从而使集群的实际状态符合所需的状态。

### Kubernetes Master


Kubernetes master 负责维护您集群的所需状态。当您与 Kubernetes 进行交互时（例如使用 `kubectl` 命令行界面），您就是在与集群的 Kubernetes master 进行通信。


> "master" 是指管理集群状态的进程的集合。通常，这些进程都运行在集群中的单个节点上，并且该节点也被称为 master 节点。您也可以通过实现多个 master 副本来提供高可用。

### Kubernetes Nodes


集群中的 node 是运行应用程序和云工作流的机器（虚拟机，物理服务器等）。Kubernetes master 控制每个 node; 您很少会直接与 node 进行交互。


#### 对象元数据


* [注解（Annotations）](/docs/concepts/overview/working-with-objects/annotations/)


### 下一步


如果您想编写一个概念页面，请参阅 [使用页面模板](/docs/home/contribute/page-templates/) 来获取概念页面类型和概念模板的信息。
