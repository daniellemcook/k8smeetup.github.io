---
title: 联邦 Secret
cn-approvers:
- pigletfly
---


本指南阐述了如果在联邦控制平面中使用 secret 。

* TOC
{:toc}



## 前提条件

本指南假定您已经安装好了一个 Kubernetes 联邦集群。如果还没有安装，可以参考 [联邦管理指南](/docs/admin/federation/) 去学习如何安装一个联邦集群(或者让您的集群管理员为您做这件事)。其他的指南，比如由 Kelsey Hightower 写的 [这篇文章](https://github.com/kelseyhightower/kubernetes-cluster-federation)，也可以帮助您。

您应该对 [Kubernetes 工作知识](/docs/setup/) 有基本了解，尤其是 [Secret](/docs/concepts/configuration/secret/)。


## 概述

在联邦控制平面中的 Secret (在本指南中称为"联邦 secret") 和传统的 [Kubernetes Secret](/docs/concepts/configuration/secret/)很相似，提供了一样的功能。在联邦控制平面中创建联邦 secret 可以确保它们同步到联邦的所有集群中。


## 创建联邦 Secret

联邦 Secret 的 API 和传统的 Kubernetes Secret API 是100%兼容的。您可以通过请求联邦 apiserver 来创建一个 secret。


您可以通过使用 [kubectl](/docs/user-guide/kubectl/) 运行下面的指令来创建联邦 Secret：

``` shell
kubectl --context=federation-cluster create -f mysecret.yaml
```


`--context=federation-cluster` 参数告诉 kubectl 发送请求到联邦 apiserver 而不是某个 Kubernetes 集群。


一旦联邦 secret 被创建了，联邦控制平面就会在所有底层 Kubernetes 集群中创建匹配的 secret。
您可以通过检查底层每个集群来对其进行验证，例如：

``` shell
kubectl --context=gce-asia-east1a get secret mysecret
```

上面的命令假定您在客户端中配置了一个叫做 'gce-asia-east1a' 的上下文，用于请求那个区域的集群。

这些底层集群中的 secret 将会和联邦 secret 保持一致。


## 更新联邦 Secret

您可以像更新 Kubernetes secret 一样更新联邦 secret。但是，对于联邦 secret，您必须发送请求到联邦 apiserver 而不是某个特定的 Kubernetes 集群。联邦控制平面会确保任何时候联邦 secret 更新后，它会将对应的 secret 更新到所有的底层集群中来和它保持一致。



## 删除联邦 Secret

您可以像删除 Kubernetes secret 一样删除联邦 Secret。但是，对于联邦 secret，您必须发送请求到联邦 apiserver 而不是某个特定的 Kubernetes 集群。

例如，您可以使用 kubectl 运行下面的命令来删除联邦 secret：

```shell
kubectl --context=federation-cluster delete secret mysecret
```

要注意的是这时删除联邦 secret 并不会删除底层集群中对应的 secret。您必须自己手动删除底层集群中的 secret。
我们打算未来修复这个问题。
