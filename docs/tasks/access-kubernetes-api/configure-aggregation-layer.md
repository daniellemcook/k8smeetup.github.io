---
title: 配置汇聚层（aggregation layer）
approvers:
- lavalamp
- cheftako
- chenopis
cn-approvers:
- zhangqx2010
---



{% capture overview %}


配置 [汇聚层](/docs/concepts/api-extension/apiserver-aggregation/) 使 Kubernetes apiserver 能够扩展额外的 API（非 Kubernetes 核心 API）。

{% endcapture %}

{% capture prerequisites %}

{% include task-tutorial-prereqs.md %}


**注意：** 为了让环境中运行的汇聚层支持代理与扩展 apiserver 间的 TLS 认证，我们需要做一些配置。Kubernetes 和 kube-apiserver 有多个 CA，所以需要确保代理的签名是由汇聚层 CA 而不是其他 CA （如 master CA）完成的。

{% endcapture %}

{% capture steps %}


## 启用 apiserver 的标记


通过以下 kube-apiserver 标记启用汇聚层。这些标记必须已经被服务提供商实现。

    --requestheader-client-ca-file=<path to aggregator CA cert>
    --requestheader-allowed-names=aggregator
    --requestheader-extra-headers-prefix=X-Remote-Extra-
    --requestheader-group-headers=X-Remote-Group
    --requestheader-username-headers=X-Remote-User
    --proxy-client-cert-file=<path to aggregator proxy cert>
    --proxy-client-key-file=<path to aggregator proxy key>


如果 kube-proxy 没有和 API server 运行在同一台主机上，那么需要确保启用了如下 apiserver 标记：

    --enable-aggregator-routing=true

{% endcapture %}

{% capture whatsnext %}


* [安装扩展 api-server](/docs/tasks/access-kubernetes-api/setup-extension-api-server/) 并适配汇聚层。
* 从高层次的概览，参见 [使用汇聚层扩展 Kubernetes API](/docs/concepts/api-extension/apiserver-aggregation/)。
* 学习如何 [使用自定义资源扩展 Kubernetes API](/docs/tasks/access-kubernetes-api/extend-api-custom-resource-definitions/)。

{% endcapture %}

{% include templates/task.md %}

