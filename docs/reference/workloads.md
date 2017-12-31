---
title: 在版本 1.8 和 1.9 中工作负载（Workloads） API 的变化
cn-approvers:
- tianshapjq
approvers:
- steveperry-53
- kow3ns
---



## 概述


Kubernetes 核心工作负载（Workloads） API 类型包括 Deployment、DaemonSet、ReplicaSet 和 StatefulSet。为了给用户提供一个稳定的 API 来编排他们的工作负载，我们优先把这些类型升级到 GA 版本。而对于批处理工作负载 API （Job 和 CronJob），虽然它们也很重要，却并不在此次的计划中，它们将有其它独立的路径升级到 GA 稳定版。


- 在 1.8 版本中，我们引入了 apps/v1beta2 API 组和版本。 核心工作负载 API 的测试版本包含 Deployment、DaemonSet、ReplicaSet 和 StatefulSet 类型，如果各项反馈都不错的话，那么我们计划在 1.9 版本中升级到 GA 版本。


- 在 1.9 版本中，我们计划引入 apps/v1 组版本。我们打算将 apps/v1beta2 组版本完全升级到 apps/v1，同时弃用 apps/v1beta2。


- 我们意识到，即使在 apps/v1 发布后，用户也需要时间从 extensions/v1beta1、apps/v1beta1 和 apps/v1beta2 迁移代码。 请务必记住，弃用指南中列出的最低支持期限是最低限度。我们将继续支持组和版本之间的转换，直到用户有足够的时间进行迁移。


## 迁移


本节内容帮助用户在不同的工作负载 API 组版本间进行迁移。


### 概述


- 如果您正在使用 extensions/v1beta1 或 apps/v1beta1 组版本中的类型，则可以等到 apps/v1 组版本发布后再进行代码迁移。


- 如果您的 deployment 需要 apps/v1beta2 组版本中提供的功能，那么可以在 apps/v1 版本发布前迁移到 apps/v1beta2 组版本。


- 您需要针对最新的稳定版本编写新的代码。


- 您可以通过运行 `kubectl convert` 命令来转换版本间的 manifest 文件。


### 迁移到 apps/v1beta2


本节提供有关迁移到 apps/v1beta2 组版本的信息。它涵盖了对核心工作负载 API 类型的通用修改。对于影响特定类型的更改（例如，默认值），请参考该类型的参考文档。


#### 默认 selector 被弃用


在较早的 apps 和 extensions 组中，当核心工作负载 API 类型中的 spec.selectors 没有指定值时，将默认使用从 spec.template.metadata.labels 生成的 LabelSelector。


用户的反馈使我们确定，由于它与 strategic merge patch 和 kubectl apply 不兼容，并且一个字段的默认值从同一对象的其它字段获取，这是不符合模式的。


#### 不可变的选择器（Immutable selectors）


我们一直提醒用户注意选择器（selector）的突变。在一般情况下，核心工作负载 API 控制器并没有很好的处理选择器突变的情况。


为了提供一个一致、可用并且稳定的 API，选择器对于 apps/v1beta2 组和版本中的所有类型是不可变的。


我们相信有更好的方法来支持类似可升级的 canaries（promotable canaries） 和 精心设计的 Pod 重新标签（orchestrated Pod relabeling）等功能，但是如果受限制的选择器突变是用户必须的一个功能，那么在 GA 版本之前，我们可以在不破坏向后兼容的情况下放宽对不可变的限制。


可升级的 canaries（promotable canaries）、精心设计的 Pod 重新标签（orchestrated Pod relabeling）和受限制的选择器突变（restricted selector mutability）等功能的开发都是由用户需求驱动的。如果您需要修改核心工作负载 API 对象的选择器，请在 GitHub 的 issue 中告诉我们或者参与 SIG-apps 的讨论。


#### 默认滚动升级


在 apps/v1beta2 版本之前，有些类型将 spec.updateStrategy 默认为 RollingUpdate 以外的策略。例如，apps/v1beta1 的 StatefulSet 默认指定 OnDelete。在 apps/v1beta2 中，所有类型的 spec.updateStrategy 默认为 RollingUpdate。


#### 弃用 Created-by 注解


在 1.8 版本中弃用了 "kubernetes.io/created-by"。相反，您应该从一个对象的 ownerReferences 中指定它的 ControllerRef 来确定对象的拥有者。


## 时间表


本节详细描述核心工作负载 API 中升级和弃用类型的时间表。


### 1.8 发布版本


在 Kubernetes 1.8 版本中，我们将核心工作负载 API 类型统一在一个组和版本中。我们解决了整个 API 层面的一致性、可用性和稳定性问题。我们已经弃用了部分 apps/v1beta1 组版本和部分 extension/v1beta1 组版本，并将其替换为 apps/v1beta2 组版本。下表显示了弃用并用以替换的类型。

<table style="width:100%">
  <tr>

    <th colspan="3">已弃用</th>
    <th colspan="3">替换</th>
  </tr>
  <tr>

	<td>组（Group）</td>
    <td>版本（Version）</td>
    <td>类型（Kind）</td>
    <td>组（Group）</td>
    <td>版本（Version）</td>
    <td>类型（Kind）</td>
  </tr>
  <tr>
    <td>apps</td>
    <td>v1beta1</td>
    <td>Deployment</td>
    <td>apps</td>
    <td>v1beta2</td>
    <td>Deployment</td>
  </tr>
  <tr>
    <td>apps</td>
    <td>v1beta1</td>
    <td>ReplicaSet</td>
    <td>apps</td>
    <td>v1beta2</td>
    <td>ReplicaSet</td>
  </tr>
  <tr>
    <td>apps</td>
    <td>v1beta1</td>
    <td>StatefulSet</td>
    <td>apps</td>
    <td>v1beta2</td>
    <td>StatefulSet</td>
  </tr>
  <tr>
    <td>extensions</td>
    <td>v1beta1</td>
    <td>Deployment</td>
    <td>apps</td>
    <td>v1beta2</td>
    <td>Deployment</td>
  </tr>
  <tr>
    <td>extensions</td>
    <td>v1beta1</td>
    <td>DaemonSet</td>
    <td>apps</td>
    <td>v1beta2</td>
    <td>DaemonSet</td>
  </tr>
  <tr>
    <td>extensions</td>
    <td>v1beta1</td>
    <td>StatefulSet</td>
    <td>apps</td>
    <td>v1beta2</td>
    <td>StatefulSet</td>
  </tr>
</table>


### 1.9 发布版本


在 Kubernetes 1.9 版本中，我们的目标是解决有关 apps/v1beta2 组版本的任何反馈问题，并将组版本升级到 GA 版本。下表显示了我们计划弃用以及将取代它们的类型。

<table style="width:100%">
  <tr>

    <th colspan="3">已弃用</th>
    <th colspan="3">替换</th>
  </tr>
  <tr>

	<td>组（Group）</td>
    <td>版本（Version）</td>
    <td>类型（Kind）</td>
    <td>组（Group）</td>
    <td>版本（Version）</td>
    <td>类型（Kind）</td>
  </tr>
  <tr>
    <td>apps</td>
    <td>v1beta2</td>
    <td>Deployment</td>
    <td>apps</td>
    <td>v1</td>
    <td>Deployment</td>
  </tr>
  <tr>
    <td>apps</td>
    <td>v1beta2</td>
    <td>DaemonSet</td>
    <td>apps</td>
    <td>v1</td>
    <td>DaemonSet</td>
  </tr>
  <tr>
    <td>apps</td>
    <td>v1beta2</td>
    <td>ReplicaSet</td>
    <td>apps</td>
    <td>v1</td>
    <td>ReplicaSet</td>
  </tr>
  <tr>
    <td>apps</td>
    <td>v1beta2</td>
    <td>StatefulSet</td>
    <td>apps</td>
    <td>v1</td>
    <td>StatefulSet</td>
  </tr>
</table>


### 1.9 版本之后


因为用户将继续依赖 extensions/v1beta1、apps/v1beta1 和 apps/v1beta2 版本，所以在升级到 GA 后，我们也不会完全删除这些组版本中已弃用的类型。相反，我们将提供部分已弃用 API 层面和 GA 版本之间的自动转换功能。下表显示了我们将支持的双向转换。

<table style="width:100%">
 <tr>
    <th colspan="3">GA</th>
    <th colspan="3">Previous</th>
  </tr>
  <tr>
    <td>Group</td>
    <td>Version</td>
    <td>Kind</td>
    <td>Group</td>
    <td>Version</td>
    <td>Kind</td>
  </tr>
  <tr>
    <td rowspan="3">apps</td>
    <td rowspan="3">v1</td>
    <td rowspan="3">Deployment</td>
    <td>apps</td>
    <td>v1beta1</td>
    <td>Deployment</td>
  </tr>
  <tr>
    <td>apps</td>
    <td>v1beta2</td>
    <td>Deployment</td>
  </tr>
  <tr>
    <td>extensions</td>
    <td>v1beta1</td>
    <td>Deployment</td>
  </tr>
   <tr>
    <td rowspan="2">apps</td>
    <td rowspan="2">v1</td>
    <td rowspan="2">Daemonset</td>
    <td>apps</td>
    <td>v1beta2</td>
    <td>DaemonSet</td>
  </tr>
  <tr>
    <td>extensions</td>
    <td>v1beta1</td>
    <td>DaemonSet</td>
  </tr>
   <tr>
    <td rowspan="3">apps</td>
    <td rowspan="3">v1</td>
    <td rowspan="3">ReplicaSet</td>
    <td>apps</td>
    <td>v1beta1</td>
    <td>ReplicaSet</td>
  </tr>
  <tr>
    <td>apps</td>
    <td>v1beta2</td>
    <td>ReplicaSet</td>
  </tr>
  <tr>
    <td>extensions</td>
    <td>v1beta1</td>
    <td>ReplicaSet</td>
  </tr>
   <tr>
    <td rowspan="2">apps</td>
    <td rowspan="2">v1</td>
    <td rowspan="2">StatefulSet</td>
    <td>apps</td>
    <td>v1beta1</td>
    <td>StatefulSet</td>
  </tr>
  <tr>
    <td>apps</td>
    <td>v1beta2</td>
    <td>StatefulSet</td>
  </tr>
</table>
