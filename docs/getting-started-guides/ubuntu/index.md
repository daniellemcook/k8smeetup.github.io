---
title: Kubernetes on Ubuntu
cn-approvers:
- chentao1596
---



{% capture overview %}

使用 Ubuntu 运行 Kubernetes 集群有多种方法。本文阐述了如何在多个公有和私有云、以及裸机的 Ubuntu 上部署 Kubernetes。
{% endcapture %}

{% capture body %}

## Ubuntu 官方指南

- [Kubernetes 官方发布版本](https://www.ubuntu.com/cloud/kubernetes)



支持 AWS、GCE、Azure、Joyent、OpenStack、VMWare、Bare Metal 和本地部署。


### 快速开始


[conjure-up](http://conjure-up.io/) 为多种云和裸机提供了在 Ubuntu 上部署 Kubernetes 的最快方式。它提供了一个用户友好的界面，提示您输入云凭证和配置选项。


可供 Ubuntu 16.04 及更高版本使用：


```
sudo snap install conjure-up --classic
# 如果您刚刚才安装 Snap，那么可能需要重新登录
conjure-up kubernetes
```


也可以用于 macOS 的 Homebrew：

```
brew install conjure-up
conjure-up kubernetes
```


### 操作指南


以下是为那些在生产中选择运行 Kubernetes 的用户提供的更深入的指南：

  - [安装](/docs/getting-started-guides/ubuntu/installation)
  - [验证](/docs/getting-started-guides/ubuntu/validation)
  - [备份](/docs/getting-started-guides/ubuntu/backups)
  - [升级](/docs/getting-started-guides/ubuntu/upgrades)
  - [伸缩](/docs/getting-started-guides/ubuntu/scaling)
  - [日志](/docs/getting-started-guides/ubuntu/logging)
  - [监控](/docs/getting-started-guides/ubuntu/monitoring)
  - [网络](/docs/getting-started-guides/ubuntu/networking)
  - [安全](/docs/getting-started-guides/ubuntu/security)
  - [存储](/docs/getting-started-guides/ubuntu/storage)
  - [故障排查](/docs/getting-started-guides/ubuntu/troubleshooting)
  - [清理](/docs/getting-started-guides/ubuntu/decommissioning)
  - [操作建议](/docs/getting-started-guides/ubuntu/operational-considerations)
  - [术语](/docs/getting-started-guides/ubuntu/glossary)


## 开发人员指南

  - [本地主机使用 LXD ](/docs/getting-started-guides/ubuntu/local)



## 哪里能找到我们


我们通常参与以下 Slack 频道：

- [sig-cluster-lifecycle](https://kubernetes.slack.com/messages/sig-cluster-lifecycle/)
- [sig-cluster-ops](https://kubernetes.slack.com/messages/sig-cluster-ops/)
- [sig-onprem](https://kubernetes.slack.com/messages/sig-onprem/)


同时，我们也会关注 Kubernetes 邮件列表。
{% endcapture %}

{% include templates/concept.md %}
