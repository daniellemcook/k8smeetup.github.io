---
cn-approvers:
- tianshapjq
approvers:
- mikedanese
- luxas
- jbeda
title: kubeadm token
---

{% capture overview %}


如 [使用 bootstrap tokens 进行验证] 中所述，Bootstrap tokens 用于建立待加入集群的 node 和 master 之间的双向信任机制。


`kubeadm init` 创建一个24小时 TTL 的初始 token。以下命令允许您管理这样一个 token，您也可以创建和管理新的 token。

{% endcapture %}

{% capture body %}

## 创建 kubeadm token {#cmd-token-create}
{% include_relative generated/kubeadm_token_create.md %}


## 删除 kubeadm token {#cmd-token-delete}
{% include_relative generated/kubeadm_token_delete.md %}


## 生成 kubeadm token {#cmd-token-generate}
{% include_relative generated/kubeadm_token_generate.md %}


## 展示 kubeadm token
{% include_relative generated/kubeadm_token_list.md %}
{% endcapture %}

{% capture whatsnext %}

* [kubeadm join](kubeadm-join.md) 启动一个 Kubernetes 工作 node 并且将其加入到集群中
{% endcapture %}

{% include templates/concept.md %}
