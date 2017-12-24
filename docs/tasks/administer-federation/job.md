---
title: 联邦 Job
cn-approvers:
- pigletfly
---


{% capture overview %}


本指南阐述了如何在联邦控制平面中使用 job 。

在联邦控制平面中的 Job (在本指南中称为"联邦 job") 和传统的 [Kubernetes Job](/docs/concepts/workloads/controllers/job/)很相似，提供了一样的功能。在联邦控制平面中创建联邦 job 可以确保在所有注册的集群中 job 的并行和完成数量和预期的一致。

{% endcapture %}

{% capture prerequisites %}

* {% include federated-task-tutorial-prereqs.md %}
* You are also expected to have a basic
[working knowledge of Kubernetes](/docs/setup/) in
general and [jobs](/docs/concepts/workloads/controllers/jobs-run-to-completion/) in particular.
{% endcapture %}

{% capture steps %}


## 创建联邦 job

联邦 job 的 API 和传统的 Kubernetes job API 是完全兼容的。您可以通过请求联邦 apiserver 来创建一个联邦 job。


您可以通过使用 [kubectl](/docs/user-guide/kubectl/) 运行下面的指令来创建联邦 job：

``` shell
kubectl --context=federation-cluster create -f myjob.yaml
```


`--context=federation-cluster` 参数告诉 kubectl 发送请求到联邦 apiserver 而不是某个 Kubernetes 集群。


一旦联邦 job 被创建了，联邦控制平面就会在所有底层 Kubernetes 集群中创建对应的 job。
您可以通过检查底层每个集群来对其进行验证，例如：

``` shell
kubectl --context=gce-asia-east1a get job myjob
```

上面的例子假定您在客户端中配置了一个叫做 'gce-asia-east1a' 的上下文，用于请求那个区域的集群。


底层集群中的 job 的并行数和完成数将会和联邦 job 的保持一致。联邦控制平面将确保联邦的所有集群都和联邦 job 有同样的并行数和完成数。


### 底层集群中 job 任务的分布


默认情况下，并行数和完成数在所有底层集群中是均匀分布的。例如：如果您有 3 个注册的集群并且用 `spec.parallelism = 9` 和 `spec.completions = 18` 参数创建了一个联邦 job，然后在这 3 个集群中每个 job 的并行数会是 `spec.parallelism = 3` ，完成数会是 `spec.completions = 6` 。

如果要修改每个集群中的并行数和完成数，您可以在联邦 job 中使用 `federation.kubernetes.io/job-preferences` 作为注解键值来修改 [ReplicaAllocationPreferences](https://github.com/kubernetes/federation/blob/{{page.githubbranch}}/apis/federation/types.go)。


## 更新联邦 job

您可以像更新 Kubernetes job 一样更新联邦 job。但是，对于联邦 job，您必须发送请求到联邦 apiserver 而不是某个特定的 Kubernetes 集群。联邦控制平面会确保任何时候联邦 job 更新后，它会将对应的 job 更新到所有的底层集群中来和它保持一致。


如果您做了包含并行数和完成数的更改，联邦控制平面将会更改底层集群中的并行数和完成数以确保它们的总数和联邦 job 期望的并行数和完成数保持一致。

## 删除联邦 job


您可以像删除 Kubernetes job 一样删除联邦 job。但是，对于联邦 job，您必须发送请求到联邦 apiserver 而不是某个特定的 Kubernetes 集群。


例如，您可以使用 kubectl 运行下面的命令来删除联邦 job：

```shell
kubectl --context=federation-cluster delete job myjob
```

**注意:**删除联邦 job 并不会删除底层集群中对应的 job。您必须自己手动删除底层集群中的 job。
{: .note}

{% endcapture %}

{% include templates/task.md %}
