---
cn-approvers:
- tianshapjq
approvers:
- bgrant0607
- lavalamp
- thockin
title: Kubernetes 弃用策略
---



Kubernetes 是一个拥有许多组件和许多贡献者的大型系统。对于这样的大型系统，其功能集会随时间不断地演变，有时也可能会删除一个功能。这可能包括删除一个 API、一个参数或者甚至是一整个功能。为了避免破坏现有用户的使用，Kubernetes 遵循一个弃用策略来移除系统的一些元素。


本文档详细描述系统各方面的弃用策略。


## 弃用部分 API


由于 Kubernetes 是一个 API 驱动的系统，为了反映对问题域的深化理解，API 也会随着时间不断演变。Kubernetes API 实际上是一个 API 集合，称作 "API 组"，并且每个 API 组都是独立的版本。[API 版本](/docs/reference/api-overview/#api-versioning) 分为3个主要部分，每个都有不同的弃用策略：


| 示例     | 规划                            |
|----------|----------------------------------|
| v1       | GA (基本可用, stable)            |
| v1beta1  | Beta (预发布)                    |
| v1alpha1 | Alpha (实验性)                   |


一个给定的 Kubernetes 发布版本支持任意数量的 API 组和 API 版本。


以下规则管理 API 元素的弃用。包括：

   * REST 资源 （也叫做 API 对象）
   * REST 资源的字段
   * 枚举或常量值
   * 组件配置结构


这些规则只适用于官方正式版本，不适用 master 的任意提交或者分支版本。


**规则 #1: 只能通过递增的 API 组版本来删除 API 元素。**


一旦把一个 API 元素添加到 API 组的某个特定版本中，那么就不能在没有规划的情况下从该版本中移除或者对它的行为做出重大的改变。


注意：由于历史原因，目前有两个 "庞大的" API 组 - "core" （没有组名）和 "extensions"。资源将会逐渐从这些遗留 API 组转移到特定领域的 API 组中。


**规则 #2: API 对象必须能够在一个发布版本的不同 API 版本中切换，而且不能有信息丢失，除非整个 REST 资源在某些版本中不存在。**


例如，一个对象可以写成 v1，然后回读为 v2 并且再转换为 v1，最终生成的 v1 资源必须和原来的 v1 相同。v2 中的表示可能与 v1 不同，但是系统知道如何在两个方向之间进行转换。另外，任何在 v2 中添加的新字段必须能够和 v1 相互转换，这意味着 v1 可能需要添加一个等效字段或将其表示为注解。


**规则 #3: 一个已规划的 API 版本可能至少要等到新的 API 版本稳定并且发布后才会弃用。**


GA API 版本能够替代另一个 GA API 版本或者 beta 和 alpha API 版本。Beta API 版本*可能不能*替代 GA API 版本。


**规则 #4: 除了每个规划中的最新 API 版本，旧的 API 版本必须在宣布弃用后继续支持一段时间，时间不少于：**

   * **GA: 1年或者2个发布版本 (选择最长的一个)**
   * **Beta: 3个月或者1个发布版本 (选择最长的一个)**
   * **Alpha: 没有要求**


这最好能够用一个例子来说明。想象有一个 Kubernetes 发布版本 X，其支持一个特定的 API 组。大约每3个月有一个新的 Kubernetes 发布版本（一年4次）。以下表格描述在一系列的后续发布版本中哪个 API 组仍被支持。


<table>
  <thead>
    <tr>
      <th>发布版本</th>
      <th>API 版本</th>
      <th>注意事项</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>X</td>
      <td>v1</td>
      <td></td>
    </tr>
    <tr>
      <td>X+1</td>
      <td>v1, v2alpha1</td>
      <td></td>
    </tr>
    <tr>
      <td>X+2</td>
      <td>v1, v2alpha2</td>
      <td>
        <ul>
           <li>移除 v2alpha1, "action required" relnote</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td>X+3</td>
      <td>v1, v2beta1</td>
      <td>
        <ul>
          <li>移除 v2alpha2, "action required" relnote</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td>X+4</td>
      <td>v1, v2beta1, v2beta2</td>
      <td>
        <ul>
          <li>弃用 v2beta1, "action required" relnote</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td>X+5</td>
      <td>v1, v2, v2beta2</td>
      <td>
        <ul>
          <li>移除 v2beta1, "action required" relnote</li>
          <li>弃用 v2beta2, "action required" relnote</li>
          <li>弃用 v1, "action required" relnote</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td>X+6</td>
      <td>v1, v2</td>
      <td>
        <ul>
          <li>移除 v2beta2, "action required" relnote</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td>X+7</td>
      <td>v1, v2</td>
      <td></td>
    </tr>
    <tr>
      <td>X+8</td>
      <td>v1, v2</td>
      <td></td>
    </tr>
    <tr>
      <td>X+9</td>
      <td>v2</td>
      <td>
        <ul>
          <li>移除 v1, "action required" relnote</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>


### REST 资源 （也叫做 API 对象）


假定现在有一个需要被弃用的 REST 资源，名为 Widget，在以上的时间线中提供在 API v1 版本中。我们在 X+1 的发布中 [记录](/docs/reference/deprecation-policy/) 并且 [宣布](https://groups.google.com/forum/#!forum/kubernetes-announce) 其弃用。那么 Widget 资源将会继续存在 API v1 版本（已被弃用）但是不在 v2alpha1 中。Widget 必须存在并且保持可用直到 X+8（包含 X+8）。只有在 X+9 发布版本中，当 API v1 已经老化，Widget 资源才会被删除，同时其行为也被移除。


### REST 资源的字段


与整个 REST 资源一样，API v1 中单独的字段必须要保持可用直到 API v1 版本被删除。但是和整个 REST 资源不同的是，API v2 可能对这个字段有不同的表达，只是这个表达要可以相互转化。例如，v1 中将要被弃用的名为 "magnitude" 的字段可能在 API v2 中命名为 "deprecatedMagnitude"。当 v1 被最终移除后，这个弃用字段才能从 v2 中移除。


### 枚举或常量值


与整个 REST 资源及其字段一样，一个在 API v1 中支持的常量必须存在并且保持可用，直到 API v1 被移除。


### 组件配置结构


组件配置在版本和管理上和 RETS 资源是一样的。


### 未来的工作


随着时间的推移，Kubernetes 将会出现更多细粒度的 API 版本，到时这些规则也需要做相应的适配。


## 弃用一个参数或 CLI


Kubernetes 系统是由几个不同的部件相互协作组成的。有时一个 Kubernetes 发布版本可能会移除这些部件的一些参数或者 CLI 命令（统称 "CLI 元素"）。这些独立的部件自然被分为两个组 - 面向用户和面向管理员的组件，这两者在弃用策略上稍有不同。除非一个参数有明显的 "alpha" 或者 "beta" 前缀，或者被文档标识为 "alpha" 或者 "beta"，否则认为它是 GA 版本。


CLI 元素实际上是系统 API 的一部分，但是它们没有像 REST API 一样进行版本化，它们的弃用规则如下：


**规则 #5a: 面向用户组件的 CLI 元素（例如 kubectl）在它们宣布弃用后至少要保持：**

   * **GA: 1年或2个发布版本 (选择最长的一个)**
   * **Beta: 3个月或者1个发布版本 (选择最长的一个)**
   * **Alpha: 没有要求**


**规则 #5b: 面向管理员组件的 CLI 元素（例如 kubelet）在它们宣布弃用后至少要保持：**

   * **GA: 6个月或者1个发布版本 (选择最长的一个)**
   * **Beta: 3个月或者1个发布版本 (选择最长的一个)**
   * **Alpha: 没有要求**


**规则 #6: 在使用弃用的 CLI 元素时必须要发出警告（可选择禁用）**


## 弃用一个功能或者行为


有时一个 Kubernetes 发布版本需要弃用一些非 API 或者 CLI 控制的系统功能或者行为。在这种情况下，弃用策略如下：


**规则 #7: 被弃用的行为在宣布弃用后至少要保持一年可用。**


这并不表示对系统的所有改变都遵循这个策略。这只适用于重大的、用户可见的行为，这些行为将会影响 Kubernetes 应用的正确性或者影响 Kubernetes 集群的管理，这些行为也在彻底清除中。


## 例外


没有策略能够覆盖所有可能的情况。本文档的策略会随着时间不断演变。实际上，存在并不是完全契合本策略的情况，或者有时本策略变成了一个严重的阻碍。这些情况需要和 SIG 或者项目负责人进行讨论，以找到最好的解决方案。需要时刻记住，Kubernetes 致力于成为一个尽最大可能不破坏用户使用的稳定的系统。例外情况将会在相关的发布公告中宣布。
