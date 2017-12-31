---
title: 联邦 ConfigMap
---


{% capture overview %}


本指南介绍了如何在联邦控制平面（Federation control plane）中使用 ConfigMap。


联邦 ConfigMap 与传统 [Kubernetes
ConfigMaps](/docs/tasks/configure-pod-container/configmap/) 非常相似且提供相同的功能。在联邦控制平面中创建它们可以确保它们在联邦的所有集群中同步。

{% endcapture %}

{% capture prerequisites %}

* {% include federated-task-tutorial-prereqs.md %}

* 您应该对 [Kubernetes 工作方式](/docs/setup/pick-right-solution/) 有基本了解，特别是 [ConfigMap](/docs/tasks/configure-pod-container/configmap/)。

{% endcapture %}

{% capture steps %}


## 创建联邦 ConfigMap


联邦 ConfigMap 的 API 100% 兼容传统 Kubernetes ConfigMap 的 API。您可以通过向联邦 apiserver 发送请求来创建 ConfigMap。


您可以运行 [kubectl](/docs/user-guide/kubectl/) 来进行创建：

``` shell
kubectl --context=federation-cluster create -f myconfigmap.yaml
```


`--context=federation-cluster` 参数告诉 kubectl 将请求提交到联邦 apiserver 而不是发送给某一个 Kubernetes 集群。


一旦创建了联邦 ConfigMap，联邦控制平面就会在所有基础集群中创建对应的 ConfigMap。您可以通过检查每个基础集群来验证这一点，例如：

``` shell
kubectl --context=gce-asia-east1a get configmap myconfigmap
```


上面的命令假设您的客户端配置了名为 'gce-asia-east1a'  的上下文，您的集群处于该区域中。


这些集群集群中的 ConfigMap 将与 联邦 ConfigMap 相匹配。



## 更新联邦 ConfigMap


您可以像更新 Kubernetes ConfigMap 一样更新联邦 ConfigMap；但是，对于联邦 ConfigMap 您必须将请求发送给联邦 apiserver 而不是发送到一个特定的 Kubernetes 集群。联邦控制平面将保证在联邦 ConfigMap 被更新后，它会将所有基础集群中与之对应的 ConfigMap 进行更新。


## 删除联邦 ConfigMap


您可以像删除 Kubernetes ConfigMap 一样删除联邦 ConfigMap；但是，对于联邦 ConfigMap 您必须将请求发送给联邦 apiserver 而不是发送到一个特定的 Kubernetes 集群。


例如，您可以使用 kubectl 运行命令：

```shell
kubectl --context=federation-cluster delete configmap
```


请注意，目前删除联邦 ConfigMap 并不会删除基础集群中对应的 ConfigMap。您必须手动删除它们。我们计划在将来解决这个问题。

{% endcapture %}

{% include templates/task.md %}
