---
cn-approvers:
- tianshapjq
approvers:
- mikedanese
- luxas
- jbeda
title: kubeadm update
---

{% capture overview %}


`kubeadm upgrade` 是一个用户友好的命令，它将复杂的升级逻辑进行封装，同时支持规划升级并实际执行升级。如果需要的话，`kubeadm upgrade` 也可以用来降级集群。


每个升级过程可能有点不同，所以我们已经分别记录了每个小升级过程。请查看这些文档以获取更详细的升级指导：


 * [1.6 升级到 1.7](/docs/tasks/administer-cluster/kubeadm-upgrade-1-7/)
 * [1.7.x 升级到 1.7.y](/docs/tasks/administer-cluster/kubeadm-upgrade-1-8/)
 * [1.7 升级到 1.8](/docs/tasks/administer-cluster/kubeadm-upgrade-1-8/)
 * [1.8.x 升级到 1.8.y](/docs/tasks/administer-cluster/kubeadm-upgrade-1-8/)

{% endcapture %}

{% capture body %}
## kubeadm upgrade plan {#cmd-upgrade-plan}
{% include_relative generated/kubeadm_upgrade_plan.md %}

## kubeadm upgrade apply  {#cmd-upgrade-apply}
{% include_relative generated/kubeadm_upgrade_apply.md %}

{% endcapture %}

{% capture whatsnext %}

* [kubeadm config](kubeadm-config.md) 如果您使用 kubeadm v1.7.x 或者更低的版本来初始化您的集群，在 `kubeadm upgrade` 之前请先执行该配置
{% endcapture %}

{% include templates/concept.md %}
