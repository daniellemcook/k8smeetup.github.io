---
title: 使用 Deployment 运行无状态应用
min-kubernetes-server-version: v1.8
cn-approvers:
- chentao1596
---


{% capture overview %}


本页演示如何使用 Kubernetes Deployment 对象运行应用。

{% endcapture %}


{% capture objectives %}


* 创建一个 nginx deployment。
* 使用 kubectl 列出关于该 deployment 的信息。
* 更新这个 deployment。

{% endcapture %}


{% capture prerequisites %}

{% include task-tutorial-prereqs.md %}

{% endcapture %}


{% capture lessoncontent %}


## 创建并探索 nginx deployment


您能够通过创建 Kubernetes Deployment 对象来运行应用，还可以在 YAML 文件中描述 Deployment。例如，这个 YAML 文件描述了运行 nginx：1.7.9 Docker 镜像的 Deployment：

{% include code.html language="yaml" file="deployment.yaml" ghlink="/docs/tasks/run-application/deployment.yaml" %}



1. 基于 YAML 文件创建 Deployment：

       kubectl apply -f https://k8s.io/docs/tasks/run-application/deployment.yaml


1. 显示有关 Deployment 的信息：

       kubectl describe deployment nginx-deployment
	   
    输出类似如下内容：

        user@computer:~/website$ kubectl describe deployment nginx-deployment
        Name:     nginx-deployment
        Namespace:    default
        CreationTimestamp:  Tue, 30 Aug 2016 18:11:37 -0700
        Labels:     app=nginx
        Annotations:    deployment.kubernetes.io/revision=1
        Selector:   app=nginx
        Replicas:   2 desired | 2 updated | 2 total | 2 available | 0 unavailable
        StrategyType:   RollingUpdate
        MinReadySeconds:  0
        RollingUpdateStrategy:  1 max unavailable, 1 max surge
        Pod Template:
          Labels:       app=nginx
          Containers:
           nginx:
            Image:              nginx:1.7.9
            Port:               80/TCP
            Environment:        <none>
            Mounts:             <none>
          Volumes:              <none>
        Conditions:
          Type          Status  Reason
          ----          ------  ------
          Available     True    MinimumReplicasAvailable
          Progressing   True    NewReplicaSetAvailable
        OldReplicaSets:   <none>
        NewReplicaSet:    nginx-deployment-1771418926 (2/2 replicas created)
        No events.
		

1. 列出通过这个 deployment 创建的 pod：

       kubectl get pods -l app=nginx

    输出类似如下内容：

        NAME                                READY     STATUS    RESTARTS   AGE
        nginx-deployment-1771418926-7o5ns   1/1       Running   0          16h
        nginx-deployment-1771418926-r18az   1/1       Running   0          16h
		

1. 显示有关 pod 的信息：

       kubectl describe pod <pod-name>


   这里 `<pod-name>` 是您其中一个 pod 的名称。


## 更新 deployment


您能够通过应用新的 YAML 文件来更新 deployment。此 YAML 文件指定应更新 deployment 以使用 nginx 1.8。

{% include code.html language="yaml" file="deployment-update.yaml" ghlink="/docs/tasks/run-application/deployment-update.yaml" %}


1. 应用新的 YAML 文件：

       kubectl apply -f https://k8s.io/docs/tasks/run-application/deployment-update.yaml


1. 查看 deployment，它使用新的名称创建了 pod，并且删除了旧的 pod：

       kubectl get pods -l app=nginx


## 通过增加复本数量来扩展应用


您能够通过应用新的 YAML 文件来增加 Deployment 中的 pod 数。这个 YAML 文件将复本设置为 4，这指定 Deployment 应该有四个pod：

{% include code.html language="yaml" file="deployment-scale.yaml" ghlink="/docs/tasks/run-application/deployment-scale.yaml" %}


1. 应用新的 YAML 文件：

       kubectl apply -f https://k8s.io/docs/tasks/run-application/deployment-scale.yaml


1. 验证 Deployment 有四个 pod：

       kubectl get pods -l app=nginx

    输出类似如下内容：

        NAME                               READY     STATUS    RESTARTS   AGE
        nginx-deployment-148880595-4zdqq   1/1       Running   0          25s
        nginx-deployment-148880595-6zgi1   1/1       Running   0          25s
        nginx-deployment-148880595-fxcez   1/1       Running   0          2m
        nginx-deployment-148880595-rwovn   1/1       Running   0          2m
		

## 删除 deployment


通过名称删除 deployment：

    kubectl delete deployment nginx-deployment


## ReplicationControllers（复本控制器）- 老的方式


创建复本应用的首选方法是使用 Deployment，它使用的是 ReplicaSet。在将 Deployment 和 ReplicaSet 添加到 Kubernetes 以前，复本应用是通过 [ReplicationController](/docs/concepts/workloads/controllers/replicationcontroller/) 配置的。

{% endcapture %}


{% capture whatsnext %}


* 了解关于 [Deployment 对象](/docs/concepts/workloads/controllers/deployment/) 的更多内容。

{% endcapture %}

{% include templates/tutorial.md %}
