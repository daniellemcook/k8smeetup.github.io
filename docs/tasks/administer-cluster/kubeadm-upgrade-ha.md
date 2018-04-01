---
reviewers:
- jamiehannaford 
- luxas
- timothysc 
- jbeda
title: 将 kubeadm HA 集群从 1.9.x 升级到 1.9.y
cn-approvers:
- chentao1596
---


{% capture overview %}


本指南用于将 `kubeadm` HA 集群从版本 1.9.x 升级到 1.9.y，其中`y > x`。术语 "`kubeadm` HA clusters" 是指由 `kubeadm` 创建的包含多于一个 master 节点的集群。要为 Kubernetes 1.9.x 版本设置 HA 集群，`kubeadm` 需要额外的手动步骤。有关如何执行此操作的说明，请参阅 [使用 kubeadm 创建 HA 集群](/docs/setup/independent/high-availability/)。此处描述的升级过程的目标是按照这些说明创建的集群。请参阅 [在 v1.8 到 v1.9 之间升级/降级 kubeadm 集群](/docs/tasks/administer-cluster/kubeadm-upgrade-1-9/) 以获取有关如何使用 `kubeadm` 创建 HA 集群的更多说明。

{% endcapture %}

{% capture prerequisites %}


在进行前：


- 为了使用这里的描述过程，您需要有一个运行 1.9.0 或者更高版本的 `kubeadm` HA 集群。
- 确保您仔细阅读 [发布说明](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.9.md)。
- 请注意，`kubeadm upgrade` 不会触及任何您的工作负载，只有 Kubernetes 内部组件。作为一种最佳做法，您应该备份对您而言重要的任何内容。例如，应该预先备份任何应用级别的状态，如应用可能依赖的数据库（如 MySQL 或 MongoDB）。
- 阅读 [在 v1.8 到 v1.9 之间升级/降级 kubeadm 集群](/docs/tasks/administer-cluster/kubeadm-upgrade-1-9/) 以了解相关的先决条件。

{% endcapture %}

{% capture steps %}


## 准备工作


开始升级之前需要做一些准备工作。首先下载与您要升级到的 Kubernetes 版本相匹配的 `kubeadm` 的版本：


```shell
# 使用最新的稳定版
# 或者手动指定已经发布的版本
export VERSION=$(curl -sSL https://dl.k8s.io/release/stable.txt) 
export ARCH=amd64 # 或者：arm、arm64、ppc64le、s390x
curl -sSL https://dl.k8s.io/release/${VERSION}/bin/linux/${ARCH}/kubeadm > /tmp/kubeadm
chmod a+rx /tmp/kubeadm
```


如有必要，将该文件复制到您主 master 的 `/tmp` 下。运行此命令以检查先决条件并确定您将收到的版本：

```shell
/tmp/kubeadm upgrade plan
```


如果满足先决条件，您将获得 kubeadm 将升级到的软件版本的摘要，如下所示：

    Upgrade to the latest stable version:

    COMPONENT            CURRENT   AVAILABLE
    API Server           v1.9.0    v1.9.2
    Controller Manager   v1.9.0    v1.9.2
    Scheduler            v1.9.0    v1.9.2
    Kube Proxy           v1.9.0    v1.9.2
    Kube DNS             1.14.5    1.14.7
    Etcd                 3.2.7     3.1.11


**警告：** 目前，kubeadm HA 集群唯一受支持的配置需要使用外部管理的 etcd 集群。作为升级的一部分，不支持升级 etcd。如有必要，您将不得不根据 [etcd 升级说明](/docs/tasks/administer-cluster/configure-upgrade-etcd/) 升级 etcd 集群，这超出了这些说明的范围。
{: .caution}


## 升级您的控制平面


以下过程必须应用于单个 master 节点，并对每个后续 master 节点顺序重复。


在使用 `kubeadm` `configmap/kubeadm-config` 启动升级之前，需要修改当前 master 主机。用当前 master 主机的名称替换对 master 主机名的任何硬件引用：

```shell
kubectl get configmap -n kube-system kubeadm-config -o yaml >/tmp/kubeadm-config-cm.yaml
sed -i 's/^\([ \t]*nodeName:\).*/\1 <CURRENT-MASTER-NAME>/' /tmp/kubeadm-config-cm.yaml
kubectl apply -f /tmp/kubeadm-config-cm.yaml --force
```


现在升级过程可以开始了。使用准备步骤中确定的目标版本并运行以下命令（出现提示时按 “y” 键）：

```shell
/tmp/kubeadm upgrade apply v<YOUR-CHOSEN-VERSION-HERE>
```


如果操作成功，您将得到如下消息：

    [upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.9.2". Enjoy!


要使用 CoreDNS 作为集群内部默认的 DNS 升级集群，请使用 `--feature-gates=CoreDNS=true` 标志调用 `kubeadm upgrade apply`。


接下来，手动升级您的 CNI 提供商。


您的容器网络接口（CNI）提供商可能有自己的升级说明。检查 [插件](/docs/concepts/cluster-administration/addons/) 页面找到您的 CNI 提供商，并查看是否需要额外的升级步骤。


**注意：** `kubeadm upgrade apply` 在辅助的 master 上初始运行时，该步骤会失败（等待重新启动的静态 pod 出现超时）。如果在一两分钟后重试，它应该成功。
{: .note}


## 升级基础软件包


此时，所有您集群中的静态 pod，例如 API Server、Controller Manager、Scheduler、Kube Proxy ，已经升级，但是基础软件，例如安装在您 node 的 OS 上的 `kubelet`、`kubectl`、`kubeadm` 仍然是旧版本。为了升级基础软件包，我们将升级它们并在所有节点上逐一重启服务：


```shell
# 使用发行版的包管理器，例如：基于 RH 系统的 'yum'
# 具体版本号，请遵守 kubeadm 的输出（见上文）
yum install -y kubelet-<NEW-K8S-VERSION> kubectl-<NEW-K8S-VERSION> kubeadm-<NEW-K8S-VERSION> kubernetes-cni-<NEW-CNI-VERSION>
systemctl restart kubelet
```


在这个例子中，是假设一个基于 _rpm_ 的系统，它使用 `yum` 安装升级后的软件。在基于 _deb_ 的系统上，它将是 `apt-get update`，然后使用 `apt-get install <PACKAGE>=<NEW-K8S-VERSION>` 安装所有软件包。


现在新版本 `kubelet` 应该在主机上运行。在相应的主机上使用以下命令验证它：

```shell
systemctl status kubelet
```


从运行 `kubectl` 命令的任何位置执行以下命令，验证升级后的节点是否可用：

```shell
kubectl get nodes
```


如果上面命令输出结果的 `STATUS` 列显示升级后的主机是 `Ready`，则可以继续（在节点获得 `Ready` 之前，您可能需要重复此操作几次）。


## 如果出现问题


如果升级失败，事后的情况取决于事情出错的阶段：


1. 如果 `/tmp/kubeadm upgrade apply` 升级集群失败，它将尝试执行回滚。因此，如果这种情况发生在第一个 master 身上，那么集群仍然完好无损的可能性很大。

   您可以再次运行 `/tmp/kubeadm upgrade apply`，因为它是幂等的，最终应确保实际状态是您声明的所需状态。您可以使用参数 `--force` 运行 `/tmp/kubeadm upgrade apply` 命令更改运行的集群为 `x.x.x --> x.x.x`，它可用于从糟糕的状态中恢复过来。


2. 如果 `/tmp/kubeadm upgrade apply` 是在其中一个辅助 master 上失败，则仍然有一个正在工作的已经升级的集群，但辅助 master 的状态有些不确定。您将不得不找出哪里出了问题，并手动加入辅助 master。如上所述，有时升级其中一个辅助 master 时，首先等待重新启动的静态 pod 失败，但在一两分钟的暂停后简单地重复该操作时会成功。

{% endcapture %}

{% include templates/task.md %}
