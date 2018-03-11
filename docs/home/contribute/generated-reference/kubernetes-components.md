---
cn-approvers:
- niuhp
title: 为 Kubernetes 组件和工具生成参考页面
---


{% capture overview %}


这个页面展示了如何使用 `update-imported-docs` 工具为 [Kubernetes](https://github.com/kubernetes/kubernetes) 和 [Federation](https://github.com/kubernetes/federation) 仓库中的工具和组件生成参考文档。

{% endcapture %}


{% capture prerequisites %}


* 您需要一台运行 Linux 或 MacOS 的机器。


* 您需要安装以下软件：

    * [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

    * [Golang](https://golang.org/doc/install)  1.9 或更高版本

    * [make](https://www.gnu.org/software/make/)

    * [gcc compiler/linker](https://gcc.gnu.org/)


* 您必须设置好 `$GOPATH` 环境变量。


* 您必须知道如何创建一个 pull request 到 GitHub 仓库。通常，这需要创建一个该仓库的分支。更多信息请查阅[创建一个文档 PR](/docs/home/contribute/create-pull-request/)。

{% endcapture %}


{% capture steps %}


## 获取两个仓库


如果您还没有 `kubernetes/website` 仓库，现在就去做吧：

```shell
mkdir $GOPATH/src
cd $GOPATH/src
go get github.com/kubernetes/website
```


确定您克隆的 [kubernetes/website](https://github.com/kubernetes/website) 仓库的根目录。例如，如果您按照前面的步骤来获取这个仓库，
您的根目录是 `$GOPATH/src/github.com/kubernetes/website.` 剩下的步骤就是将 `<web-base>` 作为您的基本目录。


如果您计划对参考文档进行更改，并且您还没有获取 `kubernetes/kubernetes` 仓库， 现在就去做吧：:

```shell
mkdir $GOPATH/src
cd $GOPATH/src
go get github.com/kubernetes/kubernetes
```


确定您克隆的 [kubernetes/kubernetes](https://github.com/kubernetes/kubernetes) 仓库的根目录。例如，如果您按照前面的步骤来获取这个仓库，
您的根目录是 `$GOPATH/src/github.com/kubernetes/kubernetes.` 剩下的步骤就是将 `<k8s-base>` 作为您的基本目录。


**注意：**
如果您仅需生成，并不更改参考文档，就不必手动获取 `kubernetes/kubernetes` 仓库了，当您运行 `update-imported-docs` 工具，它将自动克隆 `kubernetes/kubernetes` 仓库。
{: .note}


## 编辑 Kubernetes 源码


Kubernetes 组件和工具的参考文档是从 Kubernetes 源码自动生成，如果您想更改这些参考文档，第一步就是更改一处或多处 Kubernetes 源码的注释。
在您的本地 kubernetes/kubernetes 仓库做出更改，然后提交一个 PR 到 [github.com/kubernetes/kubernetes](https://github.com/kubernetes/kubernetes) 的主分支。


[PR 56942](https://github.com/kubernetes/kubernetes/pull/56942) 是一个对 Kubernetes 源码注释做出更改的 PR 示例。


监控您的 PR 并对评审者的评论做出回应。继续监控您的 PR 直到它被合并到 `kubernetes/kubernetes` 仓库的主分支。


## Cherry-pick 您的更改到已发布分支


现在您的更改已经在主分支中,它将用于下一个 Kubernetes 发布版本的开发。如果您想您的更改出现在已发布的 Kubernetes 版本中，您需要提出将您的更改挑选到到已发布分支中。


例如，假设当前的主分支是用来开发 Kubernetes 1.10 版本的，您想将您的更改补丁到已发布的 1.9 分支。了解如何做到这点，请查阅 [提出一个 Cherry Pick](https://github.com/kubernetes/community/blob/master/contributors/devel/cherry-picks.md).


监控您的 cherry-pick PR 直到它被合并到主分支。


**注意：** 提出一个 cherry pick 要求您有设置标签的权限并在您的 PR 中有一个里程碑。
如果您没有这些权限，您必须和一个能为您设置标签和里程碑的人一起工作。{: .note}


## update-imported-docs 概述


`update-imported-docs` 工具执行以下步骤：

   
1. 克隆 `kubernetes/kubernetes` 仓库。
1. 运行 `kubernetes/kubernetes/hack` 下的几个脚本。 这些脚本生成 Markdown 文件并替换 `kubernetes/kubernetes/docs` 下的文件。
1. 复制生成的 Markdown 文件到一个在 `kubernetes/website/docs/reference/generated` 下的 `kubernetes/website` 仓库的本地克隆。
1. 克隆 `kubernetes/federation` 仓库。
1. 运行 `kubernetes/federation/hack` 下的几个脚本。 这些脚本生成 Markdown 文件并替换 `kubernetes/federation/docs` 下的文件。
1. 复制生成的 Markdown 文件到一个在 `kubernetes/website/docs/reference/generated` 下的 `kubernetes/website` 仓库的本地克隆。


在这些 Markdown 文件都在您的 `kubernetes/website` 仓库的本地克隆之后，您可以在一个 
[PR](https://kubernetes.io/docs/home/contribute/create-pull-request/) 中提交他们到 `kubernetes/website`。


## 设置分支


打开 `<web-base>/update-imported-docs/config.yaml` 开始编辑。


在您想要更改的 Kubernetes 发布版本设置 `branch` 的值。例如，如果您想生成 Kubernetes 1.9 发布版本的文档，设置 `branch` 的值为 `release-1.9` 。

```shell
repos:
- name: kubernetes
  remote: https://github.com/kubernetes/kubernetes.git
  branch: release-1.9
```


## 设置源和目标


`update-imported-docs` 工具使用 `config.yaml` 中的 `src` 和 `dst` 字段来表示从 `kubernetes/kubernetes` 仓库复制的文件和要替换 `kubernetes/website` 仓库中的文件。


例如，假设你想使用这个工具复制 `kubernetes/kubernetes` 仓库中 `docs/admin` 目录下的 `kube-apiserver.md` 文件到 `kubernetes/website` 仓库中的 `docs/reference/generated/` 目录下，
您可以在 `config.yaml` 中设置 `src` 和 `dst` 字段如下所示：

```shell
repos:
- name: kubernetes
  remote: https://github.com/kubernetes/kubernetes.git
  branch: release-1.9
  files:
  - src: docs/admin/kube-apiserver.md
    dst: docs/reference/generated/kube-apiserver.md
  ...
```


这个配置和 `kubernetes/federation` 中文件的配置相似，下面是一个使用该工具复制 `kubernetes/federation` 仓库中 `docs/admin` 目录下 `kubefed_init.md` 文件到 `kubernetes/website` 仓库中 `docs/reference/generated` 目录下的配置示例：

```shell
- name: federation
  remote: https://github.com/kubernetes/federation.git
#  # Change this to a release branch when federation has release branches.
  branch: master
  files:
  - src: docs/admin/kubefed_init.md
    dst: docs/reference/generated/kubefed_init.md
  ...
```


下面是一个 `config.yaml` 文件的例子，它描述了在 Kubernetes 1.9 发布版本开始时使用 `update-imported-docs` 工具生成和复制的所有 Markdown 文件的源和目标。

```shell
repos:
- name: kubernetes
  remote: https://github.com/kubernetes/kubernetes.git
  branch: release-1.9
  files:
  - src: docs/admin/cloud-controller-manager.md
    dst: docs/reference/generated/cloud-controller-manager.md
  - src: docs/admin/kube-apiserver.md
    dst: docs/reference/generated/kube-apiserver.md
  - src: docs/admin/kube-controller-manager.md
    dst: docs/reference/generated/kube-controller-manager.md
  - src: docs/admin/kubelet.md
    dst: docs/reference/generated/kubelet.md
  - src: docs/admin/kube-proxy.md
    dst: docs/reference/generated/kube-proxy.md
  - src: docs/admin/kube-scheduler.md
    dst: docs/reference/generated/kube-scheduler.md
  - src: docs/user-guide/kubectl/kubectl.md
    dst: docs/reference/generated/kubectl/kubectl.md
- name: federation
  remote: https://github.com/kubernetes/federation.git
#  # Change this to a release branch when federation has release branches.
  branch: master
  files:
  - src: docs/admin/federation-apiserver.md
    dst: docs/reference/generated/federation-apiserver.md
  - src: docs/admin/federation-controller-manager.md
    dst: docs/reference/generated/federation-controller-manager.md
  - src: docs/admin/kubefed_init.md
    dst: docs/reference/generated/kubefed_init.md
  - src: docs/admin/kubefed_join.md
    dst: docs/reference/generated/kubefed_join.md
  - src: docs/admin/kubefed.md
    dst: docs/reference/generated/kubefed.md
  - src: docs/admin/kubefed_options.md
    dst: docs/reference/generated/kubefed_options.md
  - src: docs/admin/kubefed_unjoin.md
    dst: docs/reference/generated/kubefed_unjoin.md
  - src: docs/admin/kubefed_version.md
    dst: docs/reference/generated/kubefed_version.md
  ```


## 运行 update-imported-docs 工具


现在您的 `config.yaml` 文件包含了您的源和目标，您可以运行 `update-imported-docs` 工具：

```shell
cd <web-base>
go get ./update-imported-docs
go run update-imported-docs/update-imported-docs.go
```


## 在  kubernetes/website 中添加并提交您的更改


使用下面命令列出要生成和复制到 `kubernetes/website` 仓库中的文件：

```
cd <web-base>
git status
```


该命令输出新增加和修改的文件。例如，输出可能是这样：

```shell
...
    modified:   docs/reference/generated/cloud-controller-manager.md
    modified:   docs/reference/generated/federation-apiserver.md
    modified:   docs/reference/generated/federation-controller-manager.md
    modified:   docs/reference/generated/kube-apiserver.md
    modified:   docs/reference/generated/kube-controller-manager.md
    modified:   docs/reference/generated/kube-proxy.md
    modified:   docs/reference/generated/kube-scheduler.md
    modified:   docs/reference/generated/kubectl/kubectl.md
    modified:   docs/reference/generated/kubefed.md
    modified:   docs/reference/generated/kubefed_init.md
    modified:   docs/reference/generated/kubefed_join.md
    modified:   docs/reference/generated/kubefed_options.md
    modified:   docs/reference/generated/kubefed_unjoin.md
    modified:   docs/reference/generated/kubefed_version.md
    modified:   docs/reference/generated/kubelet.md
```


运行 `git add` 和 `git commit` 来提交这些文件。


## 创建 PR


创建一个 PR 到 `kubernetes/website` 仓库。监控您的 PR 并对评审者的评论做出回应，继续监控您的 PR 直到它被合并。


待您的 PR 被合并几分钟后, 您的更新主题将在 [发布文档](/docs/home/) 中可见。

{% endcapture %}

{% capture whatsnext %}


* [为 kubectl 命令生成参考文档](/docs/home/contribute/generated-reference/kubectl/) 
* [为 Kubernetes API 生成参考文档](/docs/home/contribute/generated-reference/kubernetes-api/)
* [为 Kubernetes Federation API 生成参考文档](/docs/home/contribute/generated-reference/federation-api/)

{% endcapture %}


{% include templates/task.md %}
