---
approvers:
- justinsb
- clove
cn-approvers:
- xiaosuiba
cn-reviewers:
- lichuqiang
title: 在 AWS EC2 上运行 Kubernetes
---


* TOC
{:toc}



## 支持的生产级工具


* [conjure-up](/docs/getting-started-guides/ubuntu/) 是 Kubernetes 的一个开源安装程序，用于在 Ubuntu 上创建本地集成 AWS 的 Kubernetes 集群。

* [Kubernetes Operations](https://github.com/kubernetes/kops) - 生产级 K8s 安装、升级和管理工具。支持在 AWS 中运行 Debian、Ubuntu、CentOS 和 RHEL。

* [CoreOS Tectonic](https://coreos.com/tectonic/) 包含开源的 [Tectonic 安装程序](https://github.com/coreos/tectonic-installer)，用于在 AWS 上的 Container Linux 节点中创建 Kubernetes 集群。

* [`kube-aws`](https://github.com/kubernetes-incubator/kube-aws) 是由 CoreOS 发起，并由 Kubernetes Incubator 维护的一个 CLI 工具 ，用于在 [Container Linux](https://coreos.com/why/) 节点上创建和管理 Kubernetes 集群，它使用 AWS 工具：EC2、CloudFormation 和 Autoscaling。

---


## 开始使用集群

### 命令行管理工具：`kubectl`

集群启动脚本会在您的工作站上生成一个 `kubernetes` 目录。此外，您也可以从 [此页面](https://github.com/kubernetes/kubernetes/releases) 下载最新的 Kubernetes 发行版。

```shell
# OS X
export PATH=<path/to/kubernetes-directory>/platforms/darwin/amd64:$PATH

# Linux
export PATH=<path/to/kubernetes-directory>/platforms/linux/amd64:$PATH
```


可以在这里找到此工具最新的文档：[kubectl 手册](/docs/user-guide/kubectl/)。

默认情况下，`kubectl` 将使用集群启动过程中生成的 `kubeconfig` 文件对 API 进行验证。
更多信息，请阅读 [kubeconfig 文件](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)。


### 示例

查看 [一个简单的 nginx 示例](/docs/tasks/run-application/run-stateless-application-deployment/) 来试用您的新集群。

"Guestbook" 应用程序是另一个开始使用 Kubernetes 的流行示例：[guestbook 示例](https://github.com/kubernetes/examples/tree/{{page.githubbranch}}/guestbook/)。

希望获取更多完整应用程序，请查看 [examples 目录](https://github.com/kubernetes/examples/tree/{{page.githubbranch}}/)。


## 伸缩集群

不支持通过 `kubectl` 添加或删除节点。您仍然可以通过调整 [自动伸缩组（Auto Scaling Group）](http://docs.aws.amazon.com/autoscaling/latest/userguide/as-manual-scaling.html) 中的 "Desired" 和 "Max" 属性来手动调整节点数量，自动伸缩组是在安装过程中创建的。


## 拆除集群

请确保您用来配置集群的环境变量已经导出，然后在 `kubernetes` 目录中执行以下脚本：

```shell
cluster/kube-down.sh
```


## 支持级别

IaaS 提供商        | 配置管理 | 操作系统            | 网络  | 文档                                          | 符合度 | 支持级别
-------------------- | ------------ | ------------- | ----------  | --------------------------------------------- | ---------| ----------------------------
AWS                  | kops         | Debian        | k8s (VPC)   | [docs](https://github.com/kubernetes/kops)    |          | 社区 ([@justinsb](https://github.com/justinsb))
AWS                  | CoreOS       | CoreOS        | flannel     | [docs](/docs/getting-started-guides/aws)      |          | 社区
AWS                  | Juju         | Ubuntu        | flannel, calico, canal     | [docs](/docs/getting-started-guides/ubuntu)      | 100%     | 社区



想要了解所有解决方案相关的支持级别信息，请查看 [解决方案表](/docs/getting-started-guides/#table-of-solutions)。


## 进一步阅读

请查看 [Kubernetes 文档](/docs/) 了解更多关于如何管理和使用 Kubernetes 集群的详细信息。
