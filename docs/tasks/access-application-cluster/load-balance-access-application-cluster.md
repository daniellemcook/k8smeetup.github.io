---
cn-approvers:
- zhangqx2010
cn-reviewers:
- chentao1596
title: 为集群中的应用提供负载均衡访问
---


{% capture overview %}


本文描述如何创建 Kubernetes Service 对象，用于对集群中运行的应用程序提供负载均衡访问。

{% endcapture %}


{% capture prerequisites %}

{% include task-tutorial-prereqs.md %}

{% endcapture %}


{% capture objectives %}


* 运行两个 Hello World 应用实例
* 创建 Service 对象
* 使用 Service 对象访问应用

{% endcapture %}


{% capture lessoncontent %}


## 为运行在两个 Pod 中的应用创建 Service

1. 在集群中运行 Hello World 应用：

       kubectl run hello-world --replicas=2 --labels="run=load-balancer-example" --image=gcr.io/google-samples/node-hello:1.0  --port=8080

1. 查看运行 Hello World 应用的两个 Pod：

       kubectl get pods --selector="run=load-balancer-example"

    输出类似于:

       NAME                           READY     STATUS    RESTARTS   AGE
       hello-world-2189936611-8fyp0   1/1       Running   0          6m
       hello-world-2189936611-9isq8   1/1       Running   0          6m

1. 查看两个 Hello World Pod 的 replica set：

       kubectl get replicasets --selector="run=load-balancer-example"

    输出类似于:

       NAME                     DESIRED   CURRENT   AGE
       hello-world-2189936611   2         2         12m

1. 创建 Service 对象用于暴露 replica set：

       kubectl expose rs <your-replica-set-name> --type="LoadBalancer" --name="example-service"

    其中 `<your-replica-set-name>` 是 replica set 的名字。

1. 查看 Service 的 IP 地址：

       kubectl get services example-service

   输出会显示 Service 的内部 IP 地址和外部 IP 地址。如果外部地址显示为 `<pending>`，重复上述命令。

   注意：如果使用 Minikube，就不会有外部 IP 地址。外部 IP 地址将会一直是 pending 状态。

       NAME              CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
       example-service   10.0.0.160   <pending>     8080/TCP   40s

1. 使用 Service 对象访问 Hello World 应用：

       curl <your-external-ip-address>:8080

   其中 `<your-external-ip-address>` 就是 Service 的外部 IP 地址。

   输出是一个来自应用的 hello 消息：

       Hello Kubernetes!

   注意： 如果使用 Minikube，使用如下命令：

       kubectl cluster-info
       kubectl describe services example-service

   输出会显示 Minikube node 的 IP 地址和 service 的 NodePort。然后使用这条命令访问应用：

       curl <minikube-node-ip-address>:<service-node-port>

   其中 `<minikube-node-ip-address>` 使用 Minikube node 的 IP 地址，`<service-node-port>` 是 service 的 NodePort 值。


## 使用 service 配置文件

除了使用 `kubectl expose` 之外，也可以使用 [service 配置文件](/docs/user-guide/services/operations) 创建 Service。

{% endcapture %}


{% capture whatsnext %}


更多信息参见 [使用 services 连接应用](/docs/concepts/services-networking/connect-applications-service/)。
{% endcapture %}

{% include templates/tutorial.md %}
