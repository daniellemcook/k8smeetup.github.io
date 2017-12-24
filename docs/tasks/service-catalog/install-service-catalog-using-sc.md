---
title: 使用 SC 安装服务目录（Service Catalog）
approvers:
- chenopis
cn-approvers:
- lichuqiang
---


{% capture overview %}
使用 [服务目录安装程序](https://github.com/GoogleCloudPlatform/k8s-service-catalog#installation) 工具可以很容易地在 Kubernetes 集群中安装和卸载服务目录。 这一 CLI 工具在您的本地环境被安装为 `sc`。

{% endcapture %}


{% capture prerequisites %}

* 理解 [服务目录](/docs/concepts/service-catalog/) 的核心概念。
* 安装 [Go 1.6 或更高版本](https://golang.org/dl/) ，并设置 `GOPATH`。
* 安装用于生成 SSL 物料的 [cfssl](https://github.com/cloudflare/cfssl) 工具。
* 服务目录要求 Kubernetes 1.7 或更高的版本。
* [安装并设置 kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) ，将其配置为连接到一个 1.7 或更高版本的 Kubernetes 集群。
* 为安装服务目录，kubectl 用户必须被绑定到 *cluster-admin* 角色。 为实现该绑定，执行以下命令：

        kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=<user-name>

{% endcapture %}


{% capture steps %}

## 在本地环境安装 `sc`

使用 `go get` 命令安装 `sc` CLI 工具：

```Go
go get github.com/GoogleCloudPlatform/k8s-service-catalog/installer/cmd/sc
```


执行上述命令后， `sc` 应当已经被安装到您的 `GOPATH/bin` 目录。


## 在 Kubernetes 集群中安装服务目录

首先，检查所有依赖项是否已安装。运行：

```shell
sc check
```


如检查成功，应返回：

```
Dependency check passed. You are good to go.
```


接下来，运行安装命令，并指定要使用的用于备份的 `storageclass`。

```shell
sc install --etcd-backup-storageclass "standard"
```


## 卸载服务目录

如果想要使用 `sc` 工具将服务目录从 Kubernetes 集群中卸载掉，执行：

```shell
sc uninstall
```

{% endcapture %}


{% capture whatsnext %}

* 查看 [服务代理（service broker）样例](https://github.com/openservicebrokerapi/servicebroker/blob/master/gettingStarted.md#sample-service-brokers)。
* 探索 [kubernetes-incubator/service-catalog](https://github.com/kubernetes-incubator/service-catalog) 项目。

{% endcapture %}


{% include templates/task.md %}
