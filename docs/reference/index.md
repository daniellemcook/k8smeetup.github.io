---
title: 参考文档
approvers:
- chenopis
cn-approvers:
- chentao1596
---



## API 参考


* [Kubernetes API 概述](/docs/reference/api-overview/) - 对 Kubernetes API 的大概描述。
* Kubernetes API 版本

  * [1.9](/docs/reference/generated/kubernetes-api/v1.9/)
  * [1.8](https://v1-8.docs.kubernetes.io/docs/reference/)
  * [1.7](https://v1-7.docs.kubernetes.io/docs/reference/)
  * [1.6](https://v1-6.docs.kubernetes.io/docs/reference/)
  * [1.5](https://v1-5.docs.kubernetes.io/docs/reference/)


## API 客户端库


想在编程语言中调用 Kubernetes API，可以使用 [客户端库](/docs/reference/client-libraries/)。官方支持的客户端库：


- [Kubernetes Go 客户端库](https://github.com/kubernetes/client-go/)
- [Kubernetes Python 客户端库](https://github.com/kubernetes-incubator/client-python)


## CLI 参考

* [kubectl](/docs/user-guide/kubectl-overview) - 主要用于运行命令和管理 Kubernetes 集群的 CLI 工具。
    * [JSONPath](/docs/user-guide/jsonpath/) -  kubectl 使用 [JSONPath 表达式](http://goessner.net/articles/JsonPath/) 的语法指南。
* [kubeadm](/docs/admin/kubeadm/) - 能够轻松提供一个安全的 Kubernetes 集群的 CLI 工具。 
* [kubefed](/docs/admin/kubefed/) - 帮助您管理联邦集群的 CLI 工具。


## 配置参考

* [kubelet](/docs/admin/kubelet/) - 每个 node 上运行的主要的 *node 代理*。Kubelet 使用了一系列的 PodSpec 确保所描述的容器运行和健康。
* [kube-apiserver](/docs/admin/kube-apiserver/) - 为 API 对象（例如 pod, service, replication controller）校验和配置数据的 REST API.
* [kube-controller-manager](/docs/admin/kube-controller-manager/) - Kubernetes 中嵌入了核心循环控制的守护进程。
* [kube-proxy](/docs/admin/kube-proxy/) - 可以进行简单的 TCP/UDP 流转发或跨后端的循环 TCP/UDP 转发。
* [kube-scheduler](/docs/admin/kube-scheduler/) - 管理可用性、性能和容量的调度程序。
* [federation-apiserver](/docs/admin/federation-apiserver/) - 联邦集群的 API 服务器。
* [federation-controller-manager](/docs/admin/federation-controller-manager/) - Kubernetes 联邦中嵌入了核心循环控制的守护进程。


## 设计文档
用于 Kubernetes 功能设计文档的存档。好的入口点是 [Kubernetes 架构](https://git.k8s.io/community/contributors/design-proposals/architecture/architecture.md) 和 [Kubernetes 设计概述](https://git.k8s.io/community/contributors/design-proposals)。
