---
approvers:
- chenopis
cn-approvers:
- xiaosuiba
layout: docsportal
css: /css/style_user_journeys.css, https://fonts.googleapis.com/icon?family=Material+Icons
js: https://use.fontawesome.com/4bcc658a89.js, https://cdnjs.cloudflare.com/ajax/libs/prefixfree/1.0.7/prefixfree.min.js

title: 媒介
track: "USERS › APPLICATION DEVELOPER › INTERMEDIATE"
---

{% assign reference_docs_url = '/docs/reference/generated/kubernetes-api/' | append: site.latest %}

{% capture overview %}

{: .note }

   本文假设您之前已经尝试过使用 Kubernetes。此时，您应该拥有与 Kubernetes 集群（本地 Minikube 或其它）交互的基本经验，并可以使用 Deployment 等 API 对象来运行应用程序。如果还没有这样的能力，您应该先查看 [新手 APP 开发人员](/docs/user-journeys/users/application-developer/foundational/){:target="_blank"} 主题。
   
在查看本页及其链接的内容后，您应该会对以下内容有更好的理解：
* 除 Deployment 外的其他 Kubernetes 工作负载
* 使 Kubernetes 应用程序符合生产条件需要做些什么
* 能够改进您工作流程的社区工具

{% endcapture %}


{% capture body %}


## 了解其它工作负载模式

随着您的 Kubernetes 用例变得更复杂，您可能会发现熟悉更多 Kubernetes 提供的工具包会有所帮助。类似 Deployment 一类的 [基本工作负载](/docs/user-journeys/users/application-developer/foundational/#section-2){:target="_blank"} 对象可以直接运行、更新和伸缩应用程序，但他们并不适用于所有情况。

以下 API 对象提供了其它工作负载类型的功能，不管他们是 *持久型* 还是 *终止型*。


#### 持久型工作负载

像 Deployment 一样，这些 API 对象在集群上无限期的运行，直到被手动终止。他们最适合长时间运行的应用程序。


* **StatefulSet** - 类似 Deployment，StatefulSet 运行您为应用程序指定的一定数量的副本。

   {: .note }
   说 Deployment 无法处理有状态的工作负载是有误导性的。使用 PersistentVolume，您可以在 Deployment 中的任何单个 Pod 的生命周期之外保留数据。
     
     然而，StatefulSet 可以提供比 Deployment 更强的“恢复”行为保证。StatefulSet 为它们的 Pod 保持一种具有粘性的（sticky）、稳定的身份标志。下表提供了一些具体的例子：

    | | Deployment | StatefulSet |
    |---|---|---|
    | **示例 Pod 名称** | `example-b1c4` | `example-0` |
    | **当 Pod 故障时** | 使用新名称 `example-a51z` 在*任意* node 上重新调度 | 使用名称 `example-0` 在相同节点上调度 |
    | **当 node 变为不可调度时** | Pod 将被调度到新节点上，并使用新的名称 | Pod 将被标记为 "Unknown"，除非强制删除 Node 对象，否则不会重新调度 |

     实际上，这意味着 StatefulSet 最适合副本（Pod）需要以强一致方式协调其工作负载的场景。确保每个 Pod 的身份标志有助于在节点变得不可访问时（[网络分区](https://en.wikipedia.org/wiki/Network_partition){:target="_blank"}），避免出现 [脑裂](https://en.wikipedia.org/wiki/Split-brain_(computing)){:target="_blank"} 的副作用。这使得 StatefulSet 非常适合像 Cassandra 或 Elasticsearch 这样的分布式数据存储。
    


* **DaemonSet** - DaemonSet 会在集群中的每个节点上连续运行，即使是新加或交换的节点。这个保证对设置集群全局行为特别有用，例如：
  * 使用类似 `fluentd` 一类的应用进行日志和监控
  * 网络代理或 [服务网格（service mesh）](https://www.linux.com/news/whats-service-mesh-and-why-do-i-need-one){:target="_blank"}


#### 终止型工作负载

与 Deployment 相反，这些 API 对象（的运行时间）是有限的。一旦指定数量的 Pod 成功完成，它们就会停止。


* **Job** —— 您可以将其用于一次性任务，如运行脚本或设置工作队列。这些任务可以顺序或并行执行。这些任务应该相对独立，因为 Job 不支持紧密通信的并行流程。[阅读更多关于 Job 模式](/docs/concepts/workloads/controllers/jobs-run-to-completion/#job-patterns){:target="_blank"}。

* **CronJob** —— 它们与 Job 类似，但允许您将其执行安排在特定时间或定期重复。您可以使用 CronJob 发送提醒电子邮件或运行备份作业。它们使用与 *crontab* 类似的语法进行设置。



#### 其它资源

有关更多信息，可以查看 [附加 Kubernetes 资源类型列表](/docs/reference/kubectl/overview/#resource-types){:target="_blank"} 以及 [API 参考文档]({{ reference_docs_url }}){:target="_blank"} 。

可能还有一些您可能觉得有用的功能没有在这里提及到，这些功能在 [完整 Kubernetes 文档](/docs/home/?path=browse){:target="_blank"} 中有介绍。


## 部署一个生产就绪的工作负载

本网站上的初学者教程（例如 [Guestbook app](/docs/tutorials/stateless-application/guestbook/){:target="_blank"}）的目的是在集群上启动并运行工作负载。这种原型设计非常适合在建立您关于 Kubernetes 的直觉！但是，为了可靠安全地将工作负载推向生产，您需要遵循一些其他最佳实践。


#### 声明式配置

您可能会通过 kubectl 与您的Kubernetes 集群进行交互。kubectl 可用于调试集群的当前状态（如检查节点数量），或修改活动 Kubernetes 对象（例如使用 `kubectl scale` 更新工作负载的副本数）。


当使用 kubectl 更新 Kubernetes 对象时，注意不同的命令对应不同的方法这点很重要：
* [纯命令式配置](/docs/tutorials/object-management-kubectl/imperative-object-management-command/){:target="_blank"}
* [使用本地配置文件的命令式配置](/docs/tutorials/object-management-kubectl/imperative-object-management-configuration/){:target="_blank"} (typically YAML)
* [使用本地配置文件的声明式配置](/docs/tutorials/object-management-kubectl/declarative-object-management-configuration/){:target="_blank"} (typically YAML)


每种方法都有优点和缺点，尽管声明式方法（如`kubectl apply -f`）可能是生产环境中最有用的。通过这些方法，您可以依靠本地 YAML 文件确保获得正真的期望状态。这还使得您可以对配置进行版本控制，对代码审查和审计跟踪很有帮助。

想要了解其他的配置最佳实践，请熟悉 [这个指南](/docs/concepts/configuration/overview/){:target="_blank"}


#### 安全

您可能熟悉*最小特权原则*——如果您在编写或使用软件时过于慷慨，权限的负面影响可能会变得失控。你会谨慎地向你的操作系统上的软件发放 `sudo` 权限吗？如果是这样在向 Kubernetes API 服务器授予工作负载权限时，您应该谨慎行事！API server 是您集群数据源的网关；它提供了读取或修改集群状态的 endpoint。


您（或您的集群运维人员）可以使用以下方式锁定 API 访问权限：
* **ServiceAccounts** - 您的 Pod 可以绑定的身份
* **RBAC** - 一种显式授予 ServiceAccount 权限的方法

想要阅读更全面的关于安全的最佳实践，请考虑查看以下主题：
* [身份验证](/docs/admin/authentication/){:target="_blank"} （他们说他们是谁？）
* [授权](/docs/admin/authorization/){:target="_blank"} （用户是否有权限去做他们要求的内容？）


#### 资源隔离和管理

如果您的工作负载在具有多个团队或项目的*多租户*环境中运行，则您的容器不一定在节点上独自运行。他们与其他您不拥有的容器共享节点资源。


即使您的集群运维人员代表您管理集群，也需要注意以下事项：
* **Namespaces**，用于隔离
* **[资源配额](/docs/concepts/policy/resource-quotas/){:target="_blank"}**，这会影响您的团队可以使用的工作负载
* **[内存](/docs/tasks/configure-pod-container/assign-memory-resource/){:target="_blank"} 和 [CPU](/docs/tasks/configure-pod-container/assign-cpu-resource/){:target="_blank"} 请求**，对于给定的 Pod 或容器
* **[监控](/docs/tasks/debug-application-cluster/resource-usage-monitoring/){:target="_blank"}**，无论是集群级别还是应用级别

这份清单可能并不全面，但许多团队都有现有的流程来处理这一切。如果情况并非如此，那么您会发现 Kubernetes 文档相当丰富。


## 使用工具改进您的开发工作流程

作为应用程序开发人员，您可能会在工作流程中遇到以下工具。


#### kubectl

`kubectl` 是一个命令行工具，可让您轻松读取或修改您的 Kubernetes 集群。它为普通操作提供方便，简短的命令，如伸缩应用程序实例和获取节点信息。kubectl 如何做到这一点？它基本上只是一个用于发起 API 请求的用户友好的包装。它使用  [client-go](https://github.com/kubernetes/client-go/#client-go){:target="_blank"} 编写（它是 Kubernetes API 的 Go 库）。

要了解最常用的 kubectl 命令，请查看 [kubectl cheatsheet](/docs/reference/kubectl/cheatsheet/){:target="_blank"}。它对以下主题进行了说明：
* [kubeconfig 文件](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/){:target="_blank"} - 您的 kubeconfig 文件告诉 kubectl 要与哪个集群通信，并且可以引用多个集群（如 dev 和 prod）。
* [各种可用的输出格式](/docs/reference/kubectl/cheatsheet/#formatting-output){:target="_blank"}(/docs/reference/kubectl/cheatsheet/#formatting-output){:target="_blank"} - 当您使用 `kubectl get` 列举关于某些 API 对象的信息时，知道这些格式是有帮助的。


* [JSONPath 输出格式](/docs/reference/kubectl/jsonpath/){:target="_blank"} -  这与上面的输出格式有关。JSONPath 对于从 `kubectl get` 输出（例如 Service URL）中解析特定的子级字段特别有用。

* [`kubectl run` vs `kubectl apply`](/docs/reference/kubectl/conventions/){:target="_blank"} - 这与前面的 [声明式配置](#声明式配置) 部分相关联。

有关 kubectl 命令及其选项的完整列表，请查看 [参考指南](/docs/reference/generated/kubectl/kubectl-commands){:target="_blank"}。


#### Helm

要利用社区中的预打包配置，您可以使用 **Helm charts**。

Helm chart 为特定应用程序（如 Jenkins 和 Postgres）打包 YAML 配置。然后，您可以使用最少的额外配置在集群上安装和运行这些应用程序。这种方法对于不需要太多定制实现逻辑的“现成”组件最为合理。

对于编写自己的 Kubernetes 应用配置，有一个 [繁荣的工具生态系统](https://docs.google.com/a/heptio.com/spreadsheets/d/1FCgqz1Ci7_VCz_wdh8vBitZ3giBtac_H8SBw4uxnrsE/edit?usp=drive_web){:target="_blank"}，您可能会觉得有用。


## 探索更多资源

#### 参考
现在您已经非常熟悉 Kubernetes，您可能会发现浏览以下参考页面非常有用。这样可能发现其他功能可能存在的高级视图：

* [常用 `kubectl` 命令](/docs/reference/kubectl/cheatsheet/){:target="_blank"}
* [Kubernetes API 参考]({{ reference_docs_url }}){:target="_blank"}
* [标准化词汇表](/docs/reference/glossary/){:target="_blank"}

此外，[Kubernetes博客](http://blog.kubernetes.io/){:target="_blank"} 通常会在 Kubernetes 设计模式和案例研究上发布有用的帖子


#### 下一步
如果您对此页面上的主题很满意，并且想要了解更多的信息，请查看以下用户旅程：
* [高级应用程序开发人员](/docs/user-journeys/users/application-developer/advanced/){:target="_blank"} - 深入探索，进入旅程的下一个阶段。
* [基础群集操作员](/docs/user-journeys/users/cluster-operator/foundational/){:target="_blank"} - 拓宽视野，探索其他旅程。
{% endcapture %}

{% include templates/user-journey-content.md %}
