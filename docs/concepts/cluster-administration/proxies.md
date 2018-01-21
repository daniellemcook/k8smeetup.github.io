---
title: Kubernetes 中的代理
cn-approvers:
- chentao1596
---


{% capture overview %}

本页描述与 Kubernetes 一起使用的代理。
{% endcapture %}

{% capture body %}


## 代理


在使用 Kubernetes 的时候，您可能会遇到下面几种不同的代理：


1.  [kubectl 代理](/docs/tasks/access-application-cluster/access-cluster/#directly-accessing-the-rest-api)：

    - 运行在用户桌面上或 pod 中
    - 本地主机地址到 Kubernetes apiserver 的代理
    - 客户端到代理使用 HTTP
    - 代理到 apiserver 使用 HTTPS
    - 定位 apiserver
    - 添加身份验证 header


1.  [apiserver 代理](/docs/tasks/access-application-cluster/access-cluster/#discovering-builtin-services)：

    - apiserver 内置的一个 bastion
    - 将群集外部的用户连接到群集 IP，否则可能无法访问
    - 运行在 apiserver 进程中
    - 客户端到代理使用 HTTPS （如果 apiserver 配置为使用 HTTP，则这里使用 HTTP）
    - 代理到目标可以使用 HTTP 或 HTTPS，这是由代理使用可用信息来选择的
    - 可用于访问 Node、Pod 或者 Service
    - 访问 Service 时作为负载均衡
	

1.  [kube 代理](/docs/concepts/services-networking/service/#ips-and-vips)：

    - 每个 node 上都运行
    - 代理 UDP 和 TCP
    - 不支持 HTTP
    - 提供负载均衡
    - 只用来访问服务


1.  放在 apiserver 前面的代理/负载均衡：

    - 存在和实现因集群而异（例如 nginx）
    - 在所有的客户端和一个或多个 apiserver 之间
    - 如果有几个 apiserver，则充当负载均衡器。
	

1.  外部服务的云负载平衡器：

    - 由一些云服务提供商提供（例如 AWS ELB，谷歌云负载平衡器）
    - 当 Kubernetes 服务类型为 `LoadBalancer` 时，自动创建
    - 仅使用 UDP/TCP
    - 因云服务提供商的不同，实现存在差异


Kubernetes 用户通常不需要担心前两种以外的其它类型。集群管理通常会确保后面类型的设置是否正确。


## 请求重定向


代理已经取代了重定向功能。重定向已被弃用。

{% endcapture %}

{% include templates/concept.md %}
