---
cn-approvers:
- tianshapjq
title: 安全考虑
---


{% capture overview %}

默认情况下，所有提供的节点之间的所有连接（包括etcd集群）都通过 easyrsa 的 TLS 进行保护。

本文介绍已部署集群的安全注意事项和生产环境建议。
{% endcapture %}
{% capture prerequisites %}

本文假定您拥有一个使用 Juju 部署的正在运行的集群。
{% endcapture %}


{% capture steps %}

## 实现

TLS 和 easyrsa 的实现使用以下 [layers](https://jujucharms.com/docs/2.2/developer-layers)。

[layer-tls-client](https://github.com/juju-solutions/layer-tls-client)
[layer-easyrsa](https://github.com/juju-solutions/layer-easyrsa)



## 限制 ssh 访问

默认情况下，管理员可以 ss h到集群中的任意已部署节点。您可以通过以下命令来批量禁用集群节点的ssh访问权限。

    juju model-config proxy-ssh=true


注意：Juju 控制器节点在您的云中仍然有开放的 ssh 访问权限，并且在这种情况下将被用作跳板机。

有关如何管理 ssh 密钥的说明，请参阅 Juju 文档中的 [模型管理](https://jujucharms.com/docs/2.2/models) 页面。
{% endcapture %}

{% include templates/task.md %}
