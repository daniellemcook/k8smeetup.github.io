---
title: 伸缩
---


{% capture overview %}

本文将展示如何对集群上的 master 和工作 node 进行垂直伸缩。
{% endcapture %}

{% capture prerequisites %}

本文假设您已经拥有一个使用 Juju 部署的工作集群。

任何应用程序都可以在部署后进行伸缩。charm 将会更新状态信息和进度，建议运行如下命令。

```
watch -c juju status --color
```
{% endcapture %}

{% capture steps %}
## Kubernetes masters


Kubernetes master 节点充当了集群控制平面的角色。
这些 master 节点被设计为能够独立于工作节点进行伸缩，从而提高操作的灵活性。
如果想要增加 master 节点，只需要执行：

    juju add-unit kubernetes-master


这将会添加一个 master 节点到控制平面中。
参阅 [构建高可用集群](/docs/admin/high-availability) 章节以获得更多信息。


## Kubernetes 工作节点

Kubernetes 工作节点是 Kubernetes 集群中的承载单元。

默认情况下，pod 会自动分布在您部署的 Kubernetes 工作节点上。

如果想要添加更多 Kubernetes 工作节点到集群中，运行：

```
juju add-unit kubernetes-worker
```


或者指定机器限制来创建更大量的节点：

```
juju set-constraints kubernetes-worker "cpu-cores=8 mem=32G"
juju add-unit kubernetes-worker
```


参阅 [机器限制文档](https://jujucharms.com/docs/stable/charms-constraints) 来获得可能对 Kubernetes 工作节点有用的其他机器约束。

## etcd


Etcd 被用作 Kubernetes 集群的键值对存储。集群默认使用一个存储实例。

出于法定人数的原因，建议保留奇数个 etcd 节点。3、5、7或9个节点是建议的节点数量，具体取决于您的群集大小。CoreOS etcd 文档有一个图表
[最佳群集大小](https://coreos.com/etcd/docs/latest/admin_guide.html#optimal-cluster-size)，参阅以确定最佳容错数量。

```
juju add-unit etcd
```


不建议增长后收缩 etcd 集群。


## Juju 控制器


一个负责协调每台机器上的 Juju 代理（这些代理管理 Kubernetes）的节点被称为控制器节点。对于生产部署，建议启用控制器节点的 HA：

    juju enable-ha
    

启用 HA 将会创建3个控制节点，这对于大多数情况应该足够了。同时对于超大型的部署，也支持5或7个控制节点。

参阅 [Juju HA 控制器文档](https://jujucharms.com/docs/2.2/controllers-ha) 以获得更多信息。
{% endcapture %}

{% include templates/task.md %}