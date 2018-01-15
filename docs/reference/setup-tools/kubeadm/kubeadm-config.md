---
cn-approvers:
- tianshapjq
approvers:
- mikedanese
- luxas
- jbeda
title: kubeadm 配置
---

{% capture overview %}

从 v1.8.0 版本开始，kubeadm 上传集群的配置信息到 `kube-system` 命名空间下的一个名为 `kubeadm-config` 的 ConfigMap 中，然后在升级的时候读取这个 ConfigMap。
这样可以正确配置系统组件，并提供无缝的用户体验。


您可以通过执行 `kubeadm config view` 来查看这个 ConfigMap。如果您使用 kubeadm v1.7.x 或者更低的版本来初始化您的集群，那么在使用 `kubeadm upgrade` 之前，您必须使用 `kubeadm config upload` 来创建 ConfigMap。

{% endcapture %}

{% capture body %}
## kubeadm config upload from-file {#cmd-config-from-file}
{% include_relative generated/kubeadm_config_upload_from-file.md %}

## kubeadm config upload from-flags {#cmd-config-from-flags}
{% include_relative generated/kubeadm_config_upload_from-flags.md %}

## kubeadm config view {#cmd-config-view}
{% include_relative generated/kubeadm_config_view.md %}
{% endcapture %}

{% capture whatsnext %}
* [kubeadm upgrade](kubeadm-upgrade.md) to upgrade a Kubernetes cluster to a newer version
{% endcapture %}

{% include templates/concept.md %}