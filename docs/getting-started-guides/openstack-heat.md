---
approvers:
- idvoretskyi
- xsgordon
title: OpenStack Heat
cn-approvers:
- chentao1596
---


* TOC
{:toc}


## 开始使用 OpenStack


**注意：** 本指南使用的 `openstack-heat` 执行程序在 Kubernetes 1.8 版本中已经废弃，在未来的版本中它将被移除。


本指南带领您完成使用 `kube-up.sh` 将 Kubernetes 部署到 OpenStack 的步骤。包含的主要机制是 [OpenStack Heat](https://wiki.openstack.org/wiki/Heat) 以及 [SaltStack](https://git.k8s.io/kubernetes/cluster/saltbase) 分布式 Kubernetes。


默认的 OS 是 CentOS 7，尚未对其他操作系统进行测试。


本指南假设您可以访问具有以下特性的 OpenStack 集群：

- Nova
- Neutron
- Swift
- Glance
- Heat

- DNS 实例名称的解析


默认情况下，要求 4 个规格为 `m1.medium` 的实例。如果您没有可用的资源，请查看 [设置附加配置值](#设置附加配置值) 章节中有关减少集群占用的信息。


## 先决条件

如果已经安装和配置了所需的 OpenStack CLI 工具的版本，则您可以直接转到 [开始一个集群](#开始一个集群) 部分。


#### 安装 OpenStack CLI 工具

```sh
sudo pip install -U --force 'python-openstackclient==3.11.0'
sudo pip install -U --force 'python-heatclient==1.10.0'
sudo pip install -U --force 'python-swiftclient==3.3.0'
sudo pip install -U --force 'python-glanceclient==2.7.0'
sudo pip install -U --force 'python-novaclient==9.0.1'
```


#### 配置 OpenStack CLI 工具


关于 `openrc.sh` 文件的获取，请与本地 OpenStack 管理员联系。


有了该文件之后，您就可以通过键入下面命令将它 source 到您的环境变量中：

```sh
. ~/path/to/openrc.sh
```


此执行程序将会使用 [正确变量](http://docs.openstack.org/user-guide/common/cli_set_environment_variables_using_openstack_rc.html) 跟 OpenStack 交互，并且开启 Kubernetes 集群。


否则，您必须适当设置以下内容：

```sh
export OS_USERNAME=username
export OS_PASSWORD=password
export OS_TENANT_NAME=projectName
export OS_AUTH_URL=https://identityHost:portNumber/v2.0
export OS_TENANT_ID=tenantIDString
export OS_REGION_NAME=regionName
```


#### 设置附加配置值


此外，这里有一些常用的特定于此执行程序的变量，以及它们的示例值。在大多数情况下，您不必更改它们。有关选项的完整列表，请参阅下一节中的文件。

```sh
export STACK_NAME=KubernetesStack
export NUMBER_OF_MINIONS=3
export MAX_NUMBER_OF_MINIONS=3
export MASTER_FLAVOR=m1.small
export MINION_FLAVOR=m1.small
export EXTERNAL_NETWORK=public
export DNS_SERVER=8.8.8.8
export IMAGE_URL_PATH=http://cloud.centos.org/centos/7/images
export IMAGE_FILE=CentOS-7-x86_64-GenericCloud-1510.qcow2
export SWIFT_SERVER_URL=http://192.168.123.100:8080
export ENABLE_PROXY=false
```


#### 手动覆盖配置值


如果您没有设置环境变量，或者不希望它们被使用，请修改 `cluster/openstack-heat` 目录下这些文件中的变量：


- **[config-default.sh](http://releases.k8s.io/{{page.githubbranch}}/cluster/openstack-heat/config-default.sh)** 设置所有 heat 模板需要的参数。
- **[config-image.sh](http://releases.k8s.io/{{page.githubbranch}}/cluster/openstack-heat/config-image.sh)** 设置需要通过 glance 下载或者创建的 OpenStack 镜像的参数。
- **[openrc-default.sh](http://releases.k8s.io/{{page.githubbranch}}/cluster/openstack-heat/openrc-default.sh)** 设置跟 OpenStack 通信的环境变量。被 CLI 工具（heat，glance，swift，nova）使用。
- **[openrc-swift.sh](http://releases.k8s.io/{{page.githubbranch}}/cluster/openstack-heat/openrc-swift.sh)** 某些 OpenStack 设置为要求使用单独的 swift 凭证。把这些凭据放在这个文件里。


有关每个变量的功能，请参阅这些文件的内容。


## 开始一个集群


一旦安装了 OpenStack CLI 工具，并设置了 OpenStack 环境变量，请执行以下命令：

```sh
export KUBERNETES_PROVIDER=openstack-heat; curl -sS https://get.k8s.io | bash
```

或者，您可以下载一个 [Kubernetes 发布](https://github.com/kubernetes/kubernetes/releases) 的 1.3 或者更高的版本，然后解压。想要开启您的集群，请打开一个 shell 并运行：

```sh
cd kubernetes # Or whichever path you have extracted the release to
KUBERNETES_PROVIDER=openstack-heat ./cluster/kube-up.sh
```

或者，如果您正在一个签出的 Kubernetes 代码库下工作，并希望从源码进行构建/测试：

```sh
cd kubernetes # Or whatever your checkout root directory is called
make clean
make quick-release
KUBERNETES_PROVIDER=openstack-heat ./cluster/kube-up.sh
```


## 检查集群


kube-up 完成后，您的集群就应该处于运行中：

```console
./cluster/kubectl.sh get cs
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-1               Healthy   {"health": "true"}
etcd-0               Healthy   {"health": "true"}
```


您也可以列出集群中的所有节点：

```console
./cluster/kubectl.sh get nodes
NAME                            STATUS    AGE     VERSION
kubernetesstack-node-ojszyjtr   Ready     42m     v1.6.0+fff5156
kubernetesstack-node-tzotzcbp   Ready     46m     v1.6.0+fff5156
kubernetesstack-node-uah8pkju   Ready     47m     v1.6.0+fff5156
```

作一个新的集群，default 命名空间下没有 pod 和 replication controller：

```console
./cluster/kubectl.sh get pods
./cluster/kubectl.sh get replicationcontrollers
```


现在，您可以准备创建 Kubernetes 对象了。


## 使用集群


对于一个简单的测试，请执行以下命令：

```sh
./cluster/kubectl.sh run nginx --image=nginx --generator=run-pod/v1
```


很快，您就会有一个正在运行的 nginx pod：

```console
./cluster/kubectl.sh get pods
NAME      READY     STATUS    RESTARTS   AGE
nginx     1/1       Running   0          5m
```


nginx pod 处于运行中之后，请使用端口转发命令设置一个从您的机器到 pod 的代理。

```sh
./cluster/kubectl.sh port-forward nginx 8888:80
```


你现在应该可以通过 `http://localhost:8888` 看到 nginx。


有关更复杂的示例，请查看 [示例目录](https://github.com/kubernetes/examples/tree/{{page.githubbranch}}/)。


## 使用 OpenStack 管理集群


您可以使用 OpenStack CLI 工具管理集群中的节点。


首先，设置环境变量：

```sh
. cluster/openstack-heat/config-default.sh
. cluster/openstack-heat/openrc-default.sh
```


要获取有关集群的所有信息，请使用 heat：

```sh
openstack stack show $STACK_NAME
```


想要查看节点列表，请使用 nova：

```sh
nova list --name=$STACK_NAME
```


更多详细信息，请查看 [OpenStack CLI 参考](http://docs.openstack.org/cli-reference/)。

### Salt


OpenStack-Heat 执行程序使用 [脱机 Salt 配置](/docs/admin/salt/#standalone-salt-configuration-on-gce-and-others)。它只在引导主机时使用 Salt，不会创建 salt-master，也不会在节点上自动启动 salt-minion 服务。


## SSH 连接到节点


公钥在集群创建时就已经添加完成，因此，您可以很容易地通过 ssh 连接到它们进行故障排除。

```sh
ssh minion@IP_ADDRESS
```


## 集群部署定制示例

您可能会发现需要修改环境变量来改变 kube-up 的行为。以下是一些常见的场景：


#### 代理配置
如果您使用代理，并且本地环境变量已经设置好了代理，那么可以使用这些变量来设置 Kubernetes 集群：


```sh
ENABLE_PROXY=true KUBERNETES_PROVIDER=openstack-heat ./cluster/kube-up.sh
```


#### 设置不同的 Swift URL

在部署某些环境时，如果需要使用不同的 Swift URL：

```sh
 SWIFT_SERVER_URL="http://10.100.0.100:8080" KUBERNETES_PROVIDER=openstack-heat ./cluster/kube-up.sh
```


### 公网名称

有时，公网的名称需要与默认名称 `public` 不同：

```sh
EXTERNAL_NETWORK="network_external" KUBERNETES_PROVIDER=openstack-heat ./cluster/kube-up.sh
```


#### 启动额外的集群

您可能想在 OpenStack 项目中加入另一个集群。使用 `$STACK_NAME` 变量来完成这个操作。

```sh
STACK_NAME=k8s-cluster-2 KUBERNETES_PROVIDER=openstack-heat ./cluster/kube-up.sh
```


有关更多配置示例，请浏览 [配置](#设置附加配置值) 部分提到的文件。


## 卸载集群


要卸载集群，请发出以下命令：

```sh
KUBERNETES_PROVIDER=openstack-heat ./cluster/kube-down.sh
```

如果默认的 `$STACK_NAME` 修改过，您必须指定名称。请注意，卸载过程不会移除 Kubernetes 创建的任何 Cinder 卷。


## 支持级别


IaaS 供应商          | 配置管理     | OS     | 网络        | 文档                                              | 符合     | 支持级别
-------------------- | ------------ | ------ | ----------  | ---------------------------------------------     | ---------| ----------------------------
OpenStack Heat       | Saltstack    | CentOS | Neutron + flannel hostgw | [docs](/docs/getting-started-guides/openstack-heat)            |          | 社区 ([@FujitsuEnablingSoftwareTechnologyGmbH](https://github.com/FujitsuEnablingSoftwareTechnologyGmbH))


有关所有解决方案的支持级别信息，请查看 [解决方案表](/docs/getting-started-guides/#table-of-solutions)。