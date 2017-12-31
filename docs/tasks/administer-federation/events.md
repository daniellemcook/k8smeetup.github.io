---
title: 联邦 Event
cn-approvers:
- pigletfly
---



本指南阐述了如何使用联邦控制平面中的 event 来帮助调试。


## 前提条件

本指南假定您已经安装好了一个 Kubernetes 联邦集群。如果还没有安装，可以参考 [联邦管理指南](/docs/admin/federation/) 去学习如何安装一个联邦集群(或者让您的集群管理员为您做这件事)。其他的指南，比如由 Kelsey Hightower 写的 [这篇文章](https://github.com/kelseyhightower/kubernetes-cluster-federation)，也可以帮助您。

您应该对 [Kubernetes 工作知识](/docs/setup/) 有基本了解。


## 概述

在联邦控制平面中的 Event (在本指南中称为"联邦 event") 和传统的 Kubernetes Event 很相似，提供了一样的功能。联邦 Event 只存储在联邦控制平面中，不会传递到底层的 Kubernetes 集群中。

当联邦控制器处理用户表面工作的 API 资源以及它们所处的状态时，会创建 event。
您可以通过运行以下命令从联邦 apiserver 获取所有的 event：

```shell
kubectl --context=federation-cluster get events
```

标准的 kubectl get，update，delete 命令都可以工作。
