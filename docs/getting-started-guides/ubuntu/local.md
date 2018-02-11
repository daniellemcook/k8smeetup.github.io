---
cn-approvers:
- tianshapjq
title: 通过 LXD 实现 Kubernetes 本地开发
---


{% capture overview %}

在本地运行 Kubernetes 比在公有云上部署和移除集群具有明显的开发优势，如更低的成本和更快的迭代。 理想情况下，Kubernetes 开发人员可以在本地容器内产生所有必需的节点，并在提交时测试新的配置。本文将向您展示如何将集群部署到本地机器上的 LXD 容器。
{% endcapture %}


在本地机器上使用 [LXD](https://linuxcontainers.org/lxd/) 的目的是为了模拟用户在云或裸机中部署的环境。每个节点都被视为一台机器，具有与生产环境相同的特性。 每个节点都是一个单独的容器，它在里面运行 Docker 容器和 `kubectl`（更多信息请参阅 [集群简介](/docs/tutorials/kubernetes-basics/cluster-intro/)）。

{% capture prerequisites %}

安装 [conjure-up](http://conjure-up.io/)，这是一个用来部署大型软件的工具。
将当前用户添加到 `lxd` 用户组中。
    
```
sudo snap install conjure-up --classic
sudo usermod -a -G lxd $(whoami)
```


注意：如果 conjure-up 要求您在 LXD 上 "配置一个 ipv6 子网"，请选择 NO。目前还不支持在 Juju/LXD 上使用 ipv6。
{% endcapture %}

{% capture steps %}

## 部署 Kubernetes

通过以下命令启动部署：

    conjure-up kubernetes


对于本教程我们将会创建一个新的控制器 - 选择 `localhost` 云类型：

![选择云类型](/images/docs/ubuntu/00-select-cloud.png)


部署应用：

![部署应用](/images/docs/ubuntu/01-deploy.png)


等待 Juju 引导结束：

![引导](/images/docs/ubuntu/02-bootstrap.png)


等待应用被完全部署：

![等待](/images/docs/ubuntu/03-waiting.png)


执行最终的后处理步骤来自动配置 Kubernetes 环境：

![后处理](/images/docs/ubuntu/04-postprocessing.png)


查看最终的摘要信息：

![最终的摘要](/images/docs/ubuntu/05-final-summary.png)


## 访问集群

您可以通过运行以下命令来访问 Kubernetes 集群：    
    
    kubectl --kubeconfig=~/.kube/config
    

或者如果您已经运行过一次，它将创建一个新的配置文件，如摘要信息所示。    
    
    kubectl --kubeconfig=~/.kube/config.conjure-up
    
{% endcapture %}

{% include templates/task.md %}

