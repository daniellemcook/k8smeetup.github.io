---
approvers:
- chenopis
layout: docsportal
css: /css/style_user_journeys.css, https://fonts.googleapis.com/icon?family=Material+Icons
js: https://use.fontawesome.com/4bcc658a89.js, https://cdnjs.cloudflare.com/ajax/libs/prefixfree/1.0.7/prefixfree.min.js
title: 基础
track: "USERS › CLUSTER OPERATOR › FOUNDATIONAL"
---





{% capture overview %}

如果您想了解如何开始管理和运营 Kubernetes 集群，本页面和链接的主题向您介绍基础概念和任务。
本页面向您介绍 Kubernetes 集群和关键概念，以便了解和管理它。内容主要集中在集群本身，而不是在集群内运行的软件。

{% endcapture %}






{% capture body %}

## 认识 Kubernetes

如果您还没有这样做，通过阅读 [什么是Kubernetes？](/docs/concepts/overview/what-is-kubernetes/) 作为开始，它介绍了一些基本概念和术语。



Kubernetes 非常灵活，集群可以在各种各样的地方运行。您可以完全在您自己的笔记本电脑或本地开发机器上与 Kubernetes 进行交互，并在虚拟机中运行它。。Kubernetes 还可以在本地或云提供商托管的虚拟机上运行，您也可以在裸机上运行 Kubernetes 集群。



一个集群由一个或多个 [节点](/docs/concepts/architecture/nodes/) 组成；如果集群中有多个节点，则节点通过 [集群网络](/docs/concepts/cluster-administration/networking/) 连接。
无论有多少个节点，所有 Kubernetes 集群通常都具有相同的组件，在 [Kubernetes组件](/docs/concepts/overview/components) 中对此进行了描述。



## 了解 Kubernetes 基础知识

熟悉如何管理和操作 Kubernetes 好办法就是启动一套集群。
试验集群最简洁凝练的方法之一就是 [安装和使用Minikube](/docs/tasks/tools/install-minikube/)。
Minikube 是一个命令行工具，用于在本地笔记本电脑或开发计算机上的虚拟机中设置和运行单节点集群。Minikube 甚至可以通过您的浏览器在 [Katacoda Kubernetes Playground](https://www.katacoda.com/courses/kubernetes/playground) 上使用。
Katacoda 提供基于浏览器的连接到单节点集群的功能，在后台使用 minikube，以此支持大量教程来探索 Kubernetes。您也可以使用基于网络的 [与Kubernetes一起](http://labs.play-with-k8s.com/) 来达到同样的目的 - 一个可以在网上玩的临时集群。



您可以通过 dashboard，API 或使用与 Kubernetes API 交互的命令行工具（例如`kubectl`）与 Kubernetes 进行交互。
通过使用配置文件来熟悉 [组织集群访问](/docs/concepts/configuration/organize-cluster-access-kubeconfig/)。
Kubernetes API 公开了许多资源，这些资源提供用于在 Kubernetes 上运行软件的构建块和抽象。
在 [了解Kubernetes对象](/docs/concepts/overview/kubernetes-objects) 中了解关于这些资源的更多信息。
Kubernetes 文档中的许多文章都涵盖了这些资源。



* [Pod Overview](/docs/concepts/workloads/pods/pod-overview/)
  * [Pods](/docs/concepts/workloads/pods/pod/)
  * [ReplicaSets](/docs/concepts/workloads/controllers/replicaset/)
  * [Deployments](/docs/concepts/workloads/controllers/deployment/)
  * [Garbage Collection](/docs/concepts/workloads/controllers/garbage-collection/)
  * [Container Images](/docs/concepts/containers/images/)
  * [Container Environment Variables](/docs/concepts/containers/container-environment-variables/)
* [Labels and Selectors](/docs/concepts/overview/working-with-objects/labels/)
* [Namespaces](/docs/concepts/overview/working-with-objects/namespaces/)
  * [Namespaces Walkthrough](/docs/tasks/administer-cluster/namespaces-walkthrough/)
* [Services](/docs/concepts/services-networking/service/)
* [Annotations](/docs/concepts/overview/working-with-objects/annotations/)
* [ConfigMaps](/docs/tasks/configure-pod-container/configure-pod-configmap/)
* [Secrets](/docs/concepts/configuration/secret/)



作为集群管理员，您可能不需要使用这些资源，但您应该熟悉它们以便能了解集群的使用方式。
您应该了解一些附加资源，其中一些被列在 [中间资源](/docs/user-journeys/users/cluster-operator/intermediate#section-1) 下。
您也应该熟悉 [怎样管理 kubernetes 集群](/docs/concepts/cluster-administration/manage-deployment/)。



## 获取集群的有关信息

您可以 [使用 Kubernetes API 访问集群](/docs/tasks/administer-cluster/access-cluster-api/)。
如果您还不熟悉如何操作，可以查看 [入门教程](/docs/tutorials/kubernetes-basics/explore-intro/)。
使用 `kubectl`，您可以非常快速地检索 Kubernetes 集群的有关信息。
要获得有关集群中节点的基本信息，请运行命令 `kubectl get nodes`。
您可以使用命令 `kubectl describe nodes` 获取更多相同节点的详细信息。
您可以使用命令 `kubectl get componentstatuses` 来查看 kubernetes 核心的状态。



获取有关集群信息及其运行方式的其他一些资源包括：

* [监控计算，存储和网络资源的工具](/docs/tasks/debug-application-cluster/resource-usage-monitoring/)
* [核心指标管道](/docs/tasks/debug-application-cluster/core-metrics-pipeline/)
  * [指标](/docs/concepts/cluster-administration/controller-metrics/)



## 获取更多资源

### 教程

* [Kubernetes 基础](/docs/tutorials/kubernetes-basics/)
* [Kubernetes 101](/docs/user-guide/walkthrough/) - kubectl 命令行和 Pods
* [Kubernetes 201](/docs/user-guide/walkthrough/k8s201/) - labels, deployments, services, 和 health checking
* [使用 ConfigMap 配置 Redis](/docs/tutorials/configuration/configure-redis-using-configmap/)
* 无状态应用程序
  * [使用 Redis 部署 PHP 留言簿](/docs/tutorials/stateless-application/guestbook/)
  * [公开外部 IP 地址以访问应用程序](/docs/tutorials/stateless-application/expose-external-ip-address/)

{% endcapture %}

{% include templates/user-journey-content.md %}
