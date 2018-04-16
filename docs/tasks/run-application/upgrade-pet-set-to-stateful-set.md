---approvers:
- bprashanth
- enisoc
- erictune
- foxish
- janetkuo
- kow3ns
- smarterclayton
cn-approvers:
- linyouchong
title: 将 PetSet 升级为 StatefulSet
---


{% capture overview %}

本页描述如何将 PetSet（Kubernetes 1.3 或 1.4 版本）升级为 *StatefulSet* （Kubernetes 1.5 或更高版本）。
{% endcapture %}

{% capture prerequisites %}


* 如果您当前使用的集群中不存在 PetSet，或者您没有将 master 升级到 Kubernetes 1.5 或更高版本的计划，您可以跳过本任务。

{% endcapture %}

{% capture steps %}


## alpha 阶段的 PetSet 和 beta 阶段的 StatefulSet 的区别

PetSet 是在 Kubernetes 1.3 版本中引入的 alpha 资源，其在 1.5 版本中作为 beta 资源且被重命名为 StatefulSet。以下是一些值得注意的变化：


* **StatefulSet 是新的 PetSet**：在 Kubernetes 1.5 或更高版本之后，PetSet 不再可用。它变成了名为 StatefulSet 的 beta 资源。想了解为何它的名称被改变了，请查看这个[讨论帖子](https://github.com/kubernetes/kubernetes/issues/27430)。

* **StatefulSet 防止脑裂**：StatefulSet 确保一个指定序号索引最多只被分配给一个 Pod，且这个 Pod 可以在集群的任何节点上运行，这在分布式应用中可以防止脑裂的发生。*TODO: 关于这个措施的链接*

* **相反的调试注解行为**：
  在 1.5 到 1.7 版本，调试注解 (`pod.alpha.kubernetes.io/initialized`) 的默认值是 `true`。
  在 1.8 或 之后的版本，这个注解被完全忽略了，系统总是认为它被设置为 `true` 。


  缺少这个注解会造成 PetSet 暂停运行，但是不会造成 StatefulSet 暂停运行。
  多数情况下，您无需在 StatefulSet 的 manifest 中添加这个注解。


## 将 PetSet 升级到 StatefulSet


注意，以下步骤必须按顺序执行。在被告知之前，请您先不要将您的 master、node 或者 `kubectl` 升级到 Kubernetes 1.5 或更高版本。


### 找到所有的 PetSet 和它们的 manifest

首先，找到您的集群中存在的所有 PetSet：

```shell
kubectl get petsets --all-namespaces
```


如果您没有找到有 PetSet 存在，那么您可以安全地将您的集群升级到 Kubernetes 1.5 或更高版本了。


如果您找到了一些 PetSet 而且您的手头有它们的 manifest，您可以继续下一步，即准备 StatefulSet 的 manifest。


否则，您需要先将它们的 manifest 保存下来以便稍后您能够以 StatefulSet 的方式重建它们。
关于如何将已存在的 PetSet 保存到文件中，这里有一个示例命令。

```shell
# Save all existing PetSets in all namespaces into a single file. Only needed when you don't have their manifests at hand.
kubectl get petsets --all-namespaces -o yaml > all-petsets.yaml
```

### 准备 StatefulSet 的 manifest

现在，对于您手头的每个 PetSet 的 manifest，将其对应转换为 StatefulSet 的 manifest ：


1. 将 `apiVersion` 从 `apps/v1alpha1` 改为 `apps/v1beta1`。
2. 将 `kind` 从 `PetSet` 改为 `StatefulSet`。
3. 如果存在作为调试勾子的注解 `pod.alpha.kubernetes.io/initialized` 且被设置为 `true`，您可以将其删除，因为它是多余的。
   如果不存在这个注解或者存在但其被设置为 `false`，您需要了解的是在升级完成后 StatefulSet 会恢复运行。


  如果您正在升级到 1.6 或 1.7 版本，您可以将这个注解明确设置为 `false` 以保持它的暂停运行特性。
  如果您正在升级到 1.8 或更高版本，则不再存在任何注解可以让 StatefulSet 暂停运行。


建议您保存好 PetSet 的 manifest 和 StatefulSet 的 manifest，万一您决定不再升级您的集群，您还可以安全回滚并重建您的 PetSet。


### 非级联地删除所有 PetSet


如果您在上一步中发现您的集群中存在 PetSet，您需要 *非级联地* 删除所有的 PetSet。您可以在执行 `kubectl` 命令时带上 `--cascade=false` 参数实现这个目的。
需要注意的是如果没有带上这个参数，**默认会进行级联删除**，您的 PetSet 所管理的所有 Pod 将会消失。


通过指定文件名删除 PetSet。要求这些文件中只能包含 PetSet 资源，不能包含其它资源，例如 Service。

```shell
# Delete all existing PetSets without cascading
# Note that <pet-set-file> should only contain PetSets that you want to delete, but not any other resources
kubectl delete -f <pet-set-file> --cascade=false
```


或者，通过指定资源名来删除它们

```shell
# Alternatively, delete them by name and namespace without cascading
kubectl delete petsets <pet-set-name> -n=<pet-set-namespace> --cascade=false
```


确保系统中所有的 PetSet 已经被删除：

```shell
# Get all PetSets again to make sure you deleted them all
# This should return nothing
kubectl get petsets --all-namespaces
```


此时，您已经将集群中所有的 PetSet 删除，但是它们关联的 Pod、Persistent Volume 或者 Persistent Volume Claim 还继续存在。然而，由于这些 Pod 不再被 PetSet 所管理，它们很容易受到节点故障的影响，直到您完成 master 的升级且重新创建了 StatefulSet。


### 将您的 master 升级到 Kubernetes 1.5 或更高版本

现在，您可以 [升级您的 Kubernetes master](/docs/admin/cluster-management/#upgrading-a-cluster) 到 Kubernetes 1.5 或更高版本。
需要注意的是 **此时您不应该升级 Node**，因为（曾经被 PetSet 所管理的）Pod 很容易受到节点故障的影响。


### 升级 kubectl 到 Kubernetes 1.5 或更高版本
升级 `kubectl` 到 Kubernetes 1.5 或更高版本，请参考 [安装和配置 kubectl 的步骤](/docs/tasks/kubectl/install/)。


### 创建 StatefulSet

继续操作之前，确保您已经将 master 和 `kubectl` 都升级到了Kubernetes 1.5 或更高版本

```shell
kubectl version
```


输出类似如下的信息：

```shell
Client Version: version.Info{Major:"1", Minor:"5", GitVersion:"v1.5.0", GitCommit:"0776eab45fe28f02bbeac0f05ae1a203051a21eb", GitTreeState:"clean", BuildDate:"2016-11-24T22:35:03Z", GoVersion:"go1.7.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"5", GitVersion:"v1.5.0", GitCommit:"0776eab45fe28f02bbeac0f05ae1a203051a21eb", GitTreeState:"clean", BuildDate:"2016-11-24T22:30:23Z", GoVersion:"go1.7.3", Compiler:"gc", Platform:"linux/amd64"}
```


如果 `Client Version` (`kubectl` version) 和 `Server Version` (master version) 都是 1.5 或更高，您可以继续。


创建 StatefulSet 接管那些属于被删除 PetSet 的 Pod，创建时使用了上一步中生成的 StatefulSet 的 manifest 

```shell
kubectl create -f <stateful-set-file>
```


确保升级后的集群中所有 StatefulSet 已被创建且运行正常：

```shell
kubectl get statefulsets --all-namespaces
```


### 升级 node 到 Kubernetes 1.5 或更高版本（可选）


现在您可以 [升级 Kubernetes node](/docs/admin/cluster-management/#upgrading-a-cluster) 到 Kubernetes 1.5 或更高版本。这一步是可选的，但必须在全部 StatefulSet 已被创建且接管了 PetSet 的 Pod 之后进行。


为了安全地运行 StatefulSet，您需要运行版本号 >= 1.1.0 的 Node 版本。更老的版本不支持 StatefulSet 的这个特性：任何时候，最多只能有一个同样身份标识的 Pod 在集群中运行。

{% endcapture %}

{% capture whatsnext %}


了解更多关于 [伸缩一个 StatefulSet](/docs/tasks/manage-stateful-set/scale-stateful-set/)。

{% endcapture %}

{% include templates/task.md %}
