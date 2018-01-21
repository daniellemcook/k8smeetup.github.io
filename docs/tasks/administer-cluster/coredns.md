---approvers:
- johnbelamaric
title: 使用 CoreDNS 进行服务发现
最低 kubernetes server 版本: v1.9
---


{% include feature-state-alpha.md %}

{% capture overview %}

本文解释了如何启用 CoreDNS 来替换 kube-dns 进行服务发现。
{% endcapture %}

{% capture prerequisites %}
{% include task-tutorial-prereqs.md %}
{% endcapture %}

{% capture steps %}


## 使用 kubeadm 安装 CoreDNS

在 Kubernetes 1.9 版本中，[CoreDNS](https://coredns.io) 可作为 alpha 特性使用，
并可以在 `kubeadm init` 中通过设置 `CoreDNS` 特性开关为 `true` 来进行安装。


```
kubeadm init --feature-gates=CoreDNS=true
```


上面的命令安装 CoreDNS 来替换 kube-dns。

{% endcapture %}

{% capture whatsnext %}


您可以通过修改 `Corefile` 来配置 [CoreDNS](https://coredns.io)，以支持比 kube-dns 更多的用例。
想了解更多信息，请查看 [CoreDNS 网站](https://coredns.io/2017/05/08/custom-dns-entries-for-kubernetes/)。

{% endcapture %}

{% include templates/task.md %}
