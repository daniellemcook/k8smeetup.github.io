---
cn-approvers:
- lichuqiang
title: 联邦 Deployment
---


{% capture overview %}

本指南说明了如何在联邦控制平面中使用 Deployment。


联邦控制平面中的 Deployment（在本指南中称为 “联邦 Deployment”）与传统的
[Kubernetes Deployment](/docs/concepts/workloads/controllers/deployment/) 非常类似，
并提供相同的功能。 在联邦控制平面中创建联邦 Deployment 确保所需的副本数存在于注册的群集中。


**到 Kubernetes 1.5 版本为止，联邦 Deployment 还是一个 Alpha 特性。 Deployment 的核心功能已经提供，
但一些特性（例如完整的 rollout 兼容性）仍在开发中。**
{% endcapture %}

{% capture prerequisites %}

* {% include federated-task-tutorial-prereqs.md %}

* 通常您还应当拥有基本的 [Kubernetes 应用知识](/docs/setup/pick-right-solution/)，特别是 [Deployment](/docs/concepts/workloads/controllers/deployment/) 相关的应用知识。

{% endcapture %}

{% capture steps %}

## 创建联邦 Deployment

联邦 Deployment 的 API 和传统的 Kubernetes Deployment API 是兼容的。
您可以通过向联邦 apiserver 发送请求来创建一个 Deployment。


您可以通过使用 [kubectl](/docs/user-guide/kubectl/) 运行下面的指令来创建联邦 Deployment：

``` shell
kubectl --context=federation-cluster create -f mydeployment.yaml
```


`--context=federation-cluster` 参数告诉 kubectl 发送请求到联邦 apiserver 而不是某个 Kubernetes 集群。


一旦联邦 Deployment 被创建，联邦控制平面会在所有底层 Kubernetes 集群中创建一个 Deployment。
您可以通过检查底层每个集群来对其进行验证，例如：

``` shell
kubectl --context=gce-asia-east1a get deployment mydep
```


上面的命令假定您在客户端中配置了一个叫做 'gce-asia-east1a' 的上下文，用于向相应区域的集群发送请求。


底层集群中的这些 Deployment 会匹配联邦 Deployment 中副本数和修订版本（revision）相关注解_之外_的信息。
联邦控制平面确保所有集群中的副本总数与联邦 Deployment 中请求的副本数量匹配。


### 在底层集群中分布副本

默认情况下，副本会被平均分布到所有的底层集群中。 例如： 如果您有3个注册的集群并且创建了一个副本数为 9
（`spec.replicas = 9`）的联邦 Deployment，那么这 3 个集群中的每个 Deployment 都将有 3 个副本
（`spec.replicas=3`）。

为修改每个集群中的 副本数，您可以在联邦 Deployment 中以注解的形式指定 [FederatedReplicaSetPreference](https://github.com/kubernetes/federation/blob/{{page.githubbranch}}/apis/federation/types.go)，
其中注解的键为 `federation.kubernetes.io/deployment-preferences`。



## 更新联邦 Deployment

您可以像更新 Kubernetes Deployment 一样更新联邦 Deployment。 但是，对于联邦 Deployment，
您必须发送请求到联邦 apiserver 而不是某个特定的 Kubernetes 集群。
联邦控制平面会确保每当联邦 Deployment 更新时，它会更新所有底层集群中相应的 Deployment 来和更新后的内容保持一致。
所以如果（在联邦 Deployment 中）选择了滚动更新，那么底层集群会独立地进行滚动更新，并且联邦 Deployment
中的 `maxSurge` and `maxUnavailable` 只会应用于独立的集群中。 将来这种行为可能会改变。


如果您的更新包括副本数量的变化，联邦控制平面会改变底层集群中的副本数量，以确保它们的总数等于联邦 Deployment 中请求的数量。


## 删除联邦 Deployment

您可以像删除 Kubernetes Deployment 一样删除联邦 Deployment。但是，对于联邦 Deployment，您必须发送请求到联邦 apiserver 而不是某个特定的 Kubernetes 集群。


例如，您可以使用 kubectl 运行下面的命令来删除联邦 Deployment：

```shell
kubectl --context=federation-cluster delete deployment mydep
```

{% endcapture %}

{% include templates/task.md %}
