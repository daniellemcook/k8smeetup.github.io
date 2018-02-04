---
approvers:
- bprashanth
- enisoc
- erictune
- foxish
- janetkuo
- kow3ns
- smarterclayton
title: 调试 StatefulSet
cn-approvers:
- chentao1596
---


{% capture overview %}


本任务展示如何调试 StatefulSet。

{% endcapture %}

{% capture prerequisites %}


* 您需要有一个 Kubernetes 集群，必须配置 kubectl 命令行工具，用以跟您的集群进行通信。
* 您应该有一个您想要进行调试的运行中的 StatefulSet。

{% endcapture %}

{% capture steps %}


## 调试 StatefulSet


为了列出属于一个 StatefulSet 并且设置了标签 `app=myapp` 的所有 pod，您可以使用以下命令：

```shell
kubectl get pods -l app=myapp
```


如果您发现列出的任何 pod 在很长一段时间内处于 `Unknown` 或者 `Terminating` 状态，请参阅 [删除 StatefulSet Pod](/docs/tasks/manage-stateful-set/delete-pods/) 以获得有关如何处理它们的说明。您还可以使用 [调试 Pod](/docs/tasks/debug-application-cluster/debug-pod-replication-controller/) 指南去调试 StatefulSet 中的单个 pod。

{% endcapture %}

{% capture whatsnext %}


了解有关 [调试一个 init-container](/docs/tasks/debug-application-cluster/debug-init-containers/) 的更多信息。

{% endcapture %}

{% include templates/task.md %}
