---
title: 将扩展资源分配给容器
cn-approvers:
- lichuqiang
---


{% capture overview %}


本文展示了如何将扩展资源分配给容器。

{% include feature-state-stable.md %}

{% endcapture %}


{% capture prerequisites %}

{% include task-tutorial-prereqs.md %}


在进行这里的练习前，您需要先进行 [为节点发布扩展资源](/docs/tasks/administer-cluster/extended-resource-node/) 中的练习。
在该练习中，您会配置一个节点，使其发布一种 “dongle” 资源。

{% endcapture %}


{% capture steps %}


## 将扩展资源分配给 Pod

为请求扩展资源，您需要在您的容器 manifest 中包含 `resources:requests` 字段。
扩展资源完全限定于 `*.kubernetes.io/` 外的任何域中。
合法的扩展资源名称形如 `example.com/foo`，其中 `example.com` 需要替换为您的组织的域，
`foo` 是一个描述性的资源名称。


这里是拥有一个容器的 Pod 的配置文件：

{% include code.html language="yaml" file="extended-resource-pod.yaml" ghlink="/docs/tasks/configure-pod-container/extended-resource-pod.yaml" %}


在该配置文件中，您可以看到容器请求 3 个 “dongle” 资源。

创建一个 Pod：

```shell
kubectl create -f https://k8s.io/docs/tasks/configure-pod-container/extended-resource-pod.yaml
```


确认 Pod 正在运行：

```shell
kubectl get pod extended-resource-demo
```


使用 “describe” 命令查看 Pod 详情：

```shell
kubectl describe pod extended-resource-demo
```


输出展示了 “dongle” 资源的请求：

```yaml
Limits:
  example.com/dongle: 3
Requests:
  example.com/dongle: 3
```


## 尝试创建第二个 Pod

这里是拥有一个容器的 Pod 的配置文件。 容器请求两个 “dongle” 资源。

{% include code.html language="yaml" file="extended-resource-pod-2.yaml" ghlink="/docs/tasks/configure-pod-container/extended-resource-pod-2.yaml" %}


Kubernetes 将无法满足两个 “dongle” 资源的请求，因为第一个 Pod 已经占用了四个可用 “dongle” 资源中的三个。

```shell
kubectl create -f https://k8s.io/docs/tasks/configure-pod-container/extended-resource-pod-2.yaml
```


使用 “describe” 命令查看 Pod 详情：

```shell
kubectl describe pod extended-resource-demo-2
```


输出显示该 Pod 无法被调度，因为没有存在两个可用 “dongle” 资源的节点。


```
Conditions:
  Type    Status
  PodScheduled  False
...
Events:
  ...
  ... Warning   FailedScheduling  pod (extended-resource-demo-2) failed to fit in any node
fit failure summary on nodes : Insufficient example.com/dongle (1)
```


查看 Pod 状态：

```shell
kubectl get pod extended-resource-demo-2
```


输出显示该 Pod 已经被创建，但无法被调度到节点上运行，其状态为 “Pending”：

```yaml
NAME                       READY     STATUS    RESTARTS   AGE
extended-resource-demo-2   0/1       Pending   0          6m
```


## 清理

删除为此练习创建的 Pod：

```shell
kubectl delete pod extended-resource-demo-2
```

{% endcapture %}

{% capture whatsnext %}


### 针对应用开发人员

* [将内存资源分配给容器和 Pod](/docs/tasks/configure-pod-container/assign-memory-resource/)
* [将 CPU 资源分配给容器和 Pod](/docs/tasks/configure-pod-container/assign-cpu-resource/)


### 针对集群管理员

* [为节点发布扩展资源](/docs/tasks/administer-cluster/extended-resource-node/)

{% endcapture %}


{% include templates/task.md %}



