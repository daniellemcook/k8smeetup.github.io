---
cn-approvers:
- tianshapjq
title: 词汇和术语
---


{% capture overview %}

本文解释了用 Juju 部署 Kubernetes 时使用的一些术语。
{% endcapture %}

{% capture body %}



**控制器** - 云环境的管理节点。通常，每个云区域有一个控制器，而 HA 环境中有更多的控制器。控制器负责管理给定环境中的所有后续模型。它包含 Juju API 服务器及其底层数据库。


**模型** - 定义一个部署所需的一系列 charm 及其它们之间的关系。这包括主机及各种单元。一个控制器能够管理多个模型。由于管理和隔离的原因，建议将 Kubernetes 集群分成单独的模型。


**charm** - 对服务的定义，包括元数据、与其它服务的依赖关系、所需的包和应用程序管理逻辑。它包含部署 Kubernetes 集群的所有操作知识。charm 的例子包括 `kubernetes-core`、`easy-rsa`、`kibana` 和 `etcd`。


**单元** - 给定的服务实例。这些可能会（也可能不会）耗尽整个机器，并可能位于同一台机器上。例如，您可能在一台机器上运行 `kubernetes-worker`、`filebeat` 和 `topbeat` 单元，但它们是三个不同服务的不同单元。


**机器** - 一个物理节点，可以是裸机，也可以是云环境提供的虚拟机。
{% endcapture %}

{% include templates/concept.md %}
