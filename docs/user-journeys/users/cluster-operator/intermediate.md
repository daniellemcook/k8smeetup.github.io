---
approvers:
- chenopis
cn-approvers:
- xiaosuiba
layout: docsportal
css: /css/style_user_journeys.css, https://fonts.googleapis.com/icon?family=Material+Icons
js: https://use.fontawesome.com/4bcc658a89.js, https://cdnjs.cloudflare.com/ajax/libs/prefixfree/1.0.7/prefixfree.min.js
title: 媒介
track: "USERS > CLUSTER OPERATOR > INTERMEDIATE"
---


{% capture overview %}


如果您是希望扩展对 Kubernetes 掌控程度的集群运维人员，则此页面及其链接的主题扩展了在 [基础集群运维人员页面](/docs/user-journeys/users/cluster-operator/foundational) 上提供的信息。通过此页面，您可以获得有关管理整个生产集群所需的关键 Kubernetes 任务的信息。

{% endcapture %}

{% capture body %}


## 与 ingress，网络，存储和工作负载一起工作

Kubernetes 的介绍通常讨论简单的无状态应用程序。随着您进入更复杂的开发，测试和生产环境，您需要考虑更复杂的情况：


通信：Ingress 和网络

* [Ingress](/docs/concepts/services-networking/ingress/)

存储：Volume 和 PersistentVolume

* [Volumes](/docs/concepts/storage/volumes/)
* [Persistent Volumes](/docs/concepts/storage/persistent-volumes/)

工作负载

* [DaemonSets](/docs/concepts/workloads/controllers/daemonset/)
* [Stateful Sets](/docs/concepts/workloads/controllers/statefulset/)
* [Jobs](/docs/concepts/workloads/controllers/jobs-run-to-completion/)
* [CronJobs](/docs/concepts/workloads/controllers/cron-jobs/)

Pods

* [Pod 生命周期](/docs/concepts/workloads/pods/pod-lifecycle/)
  * [初始化容器](/docs/concepts/workloads/pods/init-containers/)
  * [Pod 预置](/docs/concepts/workloads/pods/podpreset/)
  * [容器生命周期挂钩](/docs/concepts/containers/container-lifecycle-hooks/)

以及 Pod 如何处理调度，优先级和中断：

* [Taint 和 Toleration](/docs/concepts/configuration/taint-and-toleration/)
* [Pod 和优先级](/docs/concepts/configuration/pod-priority-preemption/)
* [中断](/docs/concepts/workloads/pods/disruptions/)
* [分配 Pod 到 Node](/docs/concepts/configuration/assign-pod-node/)
* [管理容器的计算资源](/docs/concepts/configuration/manage-compute-resources-container/)
* [配置最佳实践](/docs/concepts/configuration/overview/)


## 实施安全最佳实践

保护集群的工作超出了 Kubernetes 本身的范围。

在 Kubernetes 中，您可以配置访问控制：

* [控制对 Kubernetes API 的访问](/docs/admin/accessing-the-api/)
* [身份认证](/docs/admin/authentication/)
* [使用准入控制器](/docs/admin/admission-controllers/)


您还可以配置授权。也就是说，您不仅可以决定用户和服务如何向 API server 进行身份认证，或者确定他们是否可以访问，还可以决定他们有权访问哪些资源。基于角色的访问控制（RBAC）是控制对 Kubernetes 资源授权的推荐机制。其它授权模式可用于更具体的使用场景。

* [授权概述](/docs/admin/authorization/)
* [使用 RBAC 授权](/docs/admin/authorization/rbac/)


您应该创建 Secrets 来保存敏感数据，例如密码，令牌或密钥。但请注意，Secret 可以提供的保护存在限制。请参阅 [Secret 文档中的风险部分](/docs/concepts/configuration/secret/#risks)。




## 实现自定义日志和监控

监视群集的健康和状态非常重要。收集指标，记录并提供对这些信息的访问是常见的需求。Kubernetes 提供了一些基本的日志记录结构，您可能需要使用其他工具来帮助汇总和分析日志数据。


从 [Kubernetes 日志基础知识](/docs/concepts/cluster-administration/logging/) 开始，了解容器如何执行日志记录和常见模式。集群运维人员通常希望添加一些东西来收集和聚合这些日志。请参阅以下主题：

* [使用 Elasticsearch 和 Kibana 记录日志](/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana/)
* [使用 Stackdriver 记录日志](/docs/tasks/debug-application-cluster/logging-stackdriver/)


与日志聚合一样，许多集群使用其他软件来帮助捕获 metrics 并显示它们。这里是相关工具的概述：[用于监视计算，存储和网络资源的工具](/docs/tasks/debug-application-cluster/resource-usage-monitoring/)。Kubernetes 还支持 [内核 metrics 管道](/docs/tasks/debug-application-cluster/core-metrics-pipeline/)，可以在水平 Pod 自动伸缩器（Horizontal Pod Autoscaler）中使用自定义 metrics。

[Prometheus](https://prometheus.io/) 是另一个 CNCF 项目，它是支持捕获和临时收集 metrics 的常用选择。安装 Prometheus 有几种方式，包括使用 [stable/prometheus](https://github.com/kubernetes/charts/tree/master/stable/prometheus) [helm](https://helm.sh/) chart，CoreOS 提供的 [prometheus operator](https://github.com/coreos/prometheus-operator) 以及 [kube-prometheus](https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus)，它增加了 Grafana 仪表板和常用配置。


[Minikube](https://github.com/kubernetes/minikube) 和某些 Kubernetes 集群的通常配置是 [与 InfluxDB 和 Grafana 一起](https://github.com/kubernetes/heapster/blob/master/docs/influxdb.md) 使用 [Heapster](https://github.com/kubernetes/heapster)。
这里有一个 [如何在集群中安装这个配置的参考](https://blog.kublr.com/how-to-utilize-the-heapster-influxdb-grafana-stack-in-kubernetes-for-monitoring-pods-4a553f4d36c9)。
从 Kubernetes 1.9 开始，[sig-instrumentation](https://github.com/kubernetes/community/tree/master/sig-instrumentation) 团队正在转变使用 heapster 进行全面监控的模式，相关细节在 [Prometheus vs. Heapster vs. Kubernetes Metrics APIs](https://brancz.com/2018/01/05/prometheus-vs-heapster-vs-kubernetes-metrics-apis/) 中进行描述。


## 其他资源

集群管理：

* [集群故障排除](/docs/tasks/debug-application-cluster/debug-cluster/)
* [调试 Pod 和 Replication Controller](/docs/tasks/debug-application-cluster/debug-pod-replication-controller/)
* [调试初始化容器](/docs/tasks/debug-application-cluster/debug-init-containers/)
* [调试 Stateful Set](/docs/tasks/debug-application-cluster/debug-stateful-set/)
* [调试应用程序](/docs/tasks/debug-application-cluster/debug-application/)
* [使用资源管理器来调查您的集群](https://github.com/kubernetes/examples/blob/master/staging/explorer/README.md)

{% endcapture %}

{% include templates/user-journey-content.md %}
