---
approvers:
- dlorenc
- r2d4
- aaron-prindle
cn-approvers:
- xiaosuiba
cn-reviewers:
- chentao1596
title: 通过 Minikube 在本地运行 Kubernetes
---



Minikube 是一个可以在本地轻松运行 Kubernetes 的工具。Minikube 会在您的笔记本电脑中的虚拟机上运行一个单节点的 Kubernetes 集群，以便用户对 Kubernetes 进行试用或者在之上进行 Kubernetes 的日常开发。

* TOC
{:toc}


### Minikube 功能

* Minikube 支持的 Kubernetes 功能包括：
  * DNS
  * NodePorts
  * ConfigMaps 和 Secrets
  * 仪表盘
  * 容器运行时：Docker，[rkt](https://github.com/rkt/rkt) 和 [CRI-O](https://github.com/kubernetes-incubator/cri-o)
  * 启用 CNI (Container Network Interface，容器网络接口)
  * Ingress


## 安装

参见 [安装 Minikube](/docs/tasks/tools/install-minikube/).


## 快速入门

这是 minikube 用法的简单演示。
如果希望改变虚拟机驱动（VM driver），请添加恰当的 `--vm-driver=xxx` 参数到 `minikube start`。Minikube 支持以下驱动：

* virtualbox
* vmwarefusion
* kvm ([driver installation](https://git.k8s.io/minikube/docs/drivers.md#kvm-driver))
* xhyve ([driver installation](https://git.k8s.io/minikube/docs/drivers.md#xhyve-driver))


请注意，下面的 IP 是动态的并且可以更改。可以通过 `minikube ip` 获取。

```shell
$ minikube start
Starting local Kubernetes cluster...
Running pre-create checks...
Creating machine...
Starting local Kubernetes cluster...

$ kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080
deployment "hello-minikube" created
$ kubectl expose deployment hello-minikube --type=NodePort
service "hello-minikube" exposed

# We have now launched an echoserver pod but we have to wait until the pod is up before curling/accessing it
# via the exposed service.
# To check whether the pod is up and running we can use the following:
$ kubectl get pod
NAME                              READY     STATUS              RESTARTS   AGE
hello-minikube-3383150820-vctvh   1/1       ContainerCreating   0          3s
# We can see that the pod is still being created from the ContainerCreating status
$ kubectl get pod
NAME                              READY     STATUS    RESTARTS   AGE
hello-minikube-3383150820-vctvh   1/1       Running   0          13s
# We can see that the pod is now Running and we will now be able to curl it:
$ curl $(minikube service hello-minikube --url)
CLIENT VALUES:
client_address=192.168.99.1
command=GET
real path=/
...
$ kubectl delete deployment hello-minikube
deployment "hello-minikube" deleted
$ minikube stop
Stopping local Kubernetes cluster...
Stopping "minikube"...
```


### 其它容器运行时

#### CRI-O

要使用 [CRI-O](https://github.com/kubernetes-incubator/cri-o) 作为容器运行时，请运行：

```bash
$ minikube start \
    --network-plugin=cni \
    --container-runtime=cri-o \
    --bootstrapper=kubeadm
```


或者可以使用扩展版本：

```bash
$ minikube start \
    --network-plugin=cni \
    --extra-config=kubelet.container-runtime=remote \
    --extra-config=kubelet.container-runtime-endpoint=/var/run/crio.sock \
    --extra-config=image-service-endpoint=/var/run/crio.sock \
    --bootstrapper=kubeadm
```


#### rkt 容器引擎

使用 [rkt](https://github.com/rkt/rkt) 作为容器运行时，请运行：

```shell
$ minikube start \
    --network-plugin=cni \
    --container-runtime=rkt
```


这将使用另一个包含 rkt 和 Docker 的 minikube ISO 镜像，并启用 CNI 网络。


### 驱动插件

如果有需要，请参阅 [DRIVERS](https://git.k8s.io/minikube/docs/drivers.md) 了解支持的驱动程序以及如何安装插件的详细说明。


### 重用 Docker 守护进程

当使用单个虚拟机运行 Kubernetes 时，重用 minikube 内置的 Docker 守护进程非常方便；因为这意味着您不必在主机上搭建一个 docker 仓库并将镜像推送到上面 —— 您可以在与 minikube 相同的 docker 守护进程中构建，以加快本地实验的速度。请确保使用 'latest' 以外的标签来标记您的 Docker 镜像，并在拉取镜像时使用这个标签。否则，如果不指定镜像的版本，那么它将被假定为 `:latest`，在对应的镜像拉取策略为 `Always` 时，这可能最终导致 `ErrImagePull` 错误，因为您在默认 docker 仓库（通常为 DockerHub）中可能还没有任何版本的 Docker 镜像。


为了能够在您的 mac/linux 主机上使用 docker 守护进程，请在 shell 中使用 `docker-env command`：

```
eval $(minikube docker-env)
```

现在您应该可以在您的 mac/linux 主机上通过命令行使用 docker 与 minikube 虚拟机中的 docker 守护进程进行通信了：

```
docker ps
```


在 Centos 7 上，docker 可能会报告以下错误：

```
Could not read CA certificate "/etc/docker/ca.pem": open /etc/docker/ca.pem: no such file or directory
```


解决的办法是更新 /etc/sysconfig/docker 以确保遵循了 minikube 的环境更改：

```
< DOCKER_CERT_PATH=/etc/docker
---
> if [ -z "${DOCKER_CERT_PATH}" ]; then
>   DOCKER_CERT_PATH=/etc/docker
> fi
```

请记得关闭 imagePullPolicy:Always 策略，否则 Kubernetes 将不会使用您在本地创建的镜像。


## 管理您的集群

### 启动集群

`minikube start` 命令可以用来启动您的集群。
这个命令创建并配置一个运行单节点 Kubernetes 集群的虚拟机。
这个命令还会配置您的 [kubectl](/docs/user-guide/kubectl-overview/) 以与此集群进行通信。

如果您处于网络代理之后，则需要通过这种方式传递代理信息：

```
https_proxy=<my proxy> minikube start --docker-env HTTP_PROXY=<my proxy> --docker-env HTTPS_PROXY=<my proxy> --docker-env NO_PROXY=192.168.99.0/24
```


不幸的是，只是设置环境变量将不起作用。

Minikube 还将创建一个 "minikube" 上下文，并将其设置为 kubectl 中的默认值。
以后如果要切换回这个上下文，请运行命令：`kubectl config use-context minikube`。


#### 指定 Kubernetes 版本

Minikube 支持运行多个不同版本的 Kubernetes。您可以像这样获取所有可用版本的列表：

```
minikube get-k8s-versions
```


您可以通过添加 `--kubernetes-version` 字符串到 `minikube start` 命令中以指示 Minikube 使用的特定的 Kubernetes 版本。例如，要运行 `v1.7.3` 版本，您可以运行以下命令：

```
minikube start --kubernetes-version v1.7.3
```


### 配置 Kubernetes

Minikube 具有一个 "configurator" 功能，允许用户使用任意值配置 Kubernetes 组件。
要使用这个功能，您可以在 `minikube start` 命令中使用 `--extra-config` 参数。



这个标志是可重复的，所以你可以通过几个不同的值来设置多个选项。

该标志是采用 `component.key=value` 形式的字符串，其中 `component` 是下面列表中的字符串之一，`key` 是配置结构体中的项，而`value` 是要设置的值。

通过检查每个组件的 Kubernetes `componentconfigs` 文档可以找到有效的 key。
以下是支持的配置的文档：

* [kubelet](https://godoc.org/k8s.io/kubernetes/pkg/apis/componentconfig#KubeletConfiguration)
* [apiserver](https://godoc.org/k8s.io/kubernetes/cmd/kube-apiserver/app/options#APIServer)
* [proxy](https://godoc.org/k8s.io/kubernetes/pkg/apis/componentconfig#KubeProxyConfiguration)
* [controller-manager](https://godoc.org/k8s.io/kubernetes/pkg/apis/componentconfig#KubeControllerManagerConfiguration)
* [etcd](https://godoc.org/github.com/coreos/etcd/etcdserver#ServerConfig)
* [scheduler](https://godoc.org/k8s.io/kubernetes/pkg/apis/componentconfig#KubeSchedulerConfiguration)


#### 示例


要在 Kubelet 上将 `MaxPods` 设置更改为 5，请传递此参数：`--extra-config=kubelet.MaxPods=5`。

该功能还支持嵌套结构。要在 scheduler 中将 `LeaderElection.LeaderElect` 设置更改为 `true`，请传递此参数：`--extra-config=scheduler.LeaderElection.LeaderElect=true`。

要将 `apiserver` 上的 `AuthorizationMode` 设置为 `RBAC`，可以使用：`--extra-config=apiserver.AuthorizationMode=RBAC`。


### 停止集群
`minikube stop` 命令可以用来停止集群。
该命令会关闭 minikube 虚拟机，但将保留所有集群状态和数据。
再次启动集群将恢复到之前的状态。


### 删除集群
`minikube delete` 命令可以用来删除集群。
该命令将关闭并删除 minikube 虚拟机。没有数据或状态会被保存下来。


## 与集群交互

### Kubectl

 `minikube start` 会创建一个名为 "minikube" 的"[kubectl context](/docs/user-guide/kubectl/{{page.version}}/#-em-set-context-em-)"。这个 context 中包含了与您的 minikube 集群进行通信的配置。


Minikube 会自动将此 context 设置为默认值，但如果将来需要切换回该值，请运行：

`kubectl config use-context minikube`,

或者像这样为每个命令传递 context：`kubectl get pods --context=minikube`。


### 仪表盘

要访问 [Kubernetes 仪表盘](/docs/tasks/access-application-cluster/web-ui-dashboard/)，请在 minikube 启动后在 shell 中运行此命令以获取访问地址：

```shell
minikube dashboard
```


### Services

要访问通过 node port 公开的服务，请在 minikube 启动后在 shell 中运行此命令以获取访问地址：

```shell
minikube service [-n NAMESPACE] [--url] NAME
```


## 网络

minikube 虚拟机通过 host-only IP 地址暴露给主机系统，可以通过 `minikube ip` 命令获取该地址。
可以通过该 IP 地址访问任何 NodePort 类型的服务。

为了确定服务的 NodePort，您可以像这样使用 `kubectl` 命令：

`kubectl get service $SERVICE --output='jsonpath="{.spec.ports[0].nodePort}"'`


## 持久化存储卷（Persistent Volumes）
Minikube 支持 `hostPath` 类型的 [PersistentVolumes](/docs/concepts/storage/persistent-volumes/)。
这些 PersistentVolume 被映射到 minikube 虚拟机内的一个目录。

Minikube 虚拟机引导到一个 tmpfs，所以大多数目录将不会在重启（`minikube stop`）后保留。

* `/data`
* `/var/lib/localkube`
* `/var/lib/docker`


下面是一个 PersistentVolume 的配置示例，用于将数据保存在 `/data` 目录中：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0001
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  hostPath:
    path: /data/pv0001/
```


## 挂载主机文件夹
一些驱动程序将在虚拟机内安装一个主机文件夹，以便您可以轻松地在虚拟机和主机之间共享文件。这些目前不可配置，而且因您正在使用的驱动程序和操作系统而有所不同。

| Driver | OS | HostFolder | VM |
| --- | --- | --- | --- |
| VirtualBox | Linux | /home | /hosthome |
| VirtualBox | OSX | /Users | /Users |
| VirtualBox | Windows | C://Users | /c/Users |
| VMWare Fusion | OSX | /Users | /Users |
| Xhyve | OSX | /Users | /Users |



## 私有容器仓库

要访问私有容器仓库，请按照 [此页面](/docs/concepts/containers/images/) 中的步骤操作。

我们推荐使用 `ImagePullSecrets`，但是如果您想在 minikube 虚拟机上配置访问权限，可以把 `.dockercfg` 放在 `/home/docker` 目录下或将 `config.json` 放在 `/home/docker/.docker` 下。


## 插件

为了让 minikube 正确启动或重启自定义插件，请在 `~/.minikube/addons` 目录中放置您想用 minikube 启动的插件。这个文件夹中的插件将被移动到 minikube 虚拟机，并在
minikube 每次启动或重启时启动。


## 通过 HTTP 代理使用 Minikube

Minikube 创建一个包含 Kubernetes 和 Docker 守护进程的虚拟机。当 Kubernetes 尝试使用 Docker 来调度容器时，Docker 守护进程可能需要访问外部网络来拉取容器。


如果您位于 HTTP 代理的后面，则可能需要向 Docker 提供代理设置。
为此，需要在 `minikube start` 期间将所需的环境变量作为参数进行传递。


示例：

```shell
$ minikube start --docker-env HTTP_PROXY=http://$YOURPROXY:PORT \
                 --docker-env HTTPS_PROXY=https://$YOURPROXY:PORT
```


如果您的虚拟机地址是 192.168.99.100，那么您的代理设置很可能会阻止 kubectl 对它的直接访问。要绕过此 IP 地址的代理配置，您应该修改 no_proxy 设置。您可以这样配置：

```shell
$ export no_proxy=$no_proxy,$(minikube ip)
```


## 已知问题
* 需要云服务提供商的功能在 Minikube 中不能使用。包括：
  * 负载均衡器（LoadBalancer）
* 需要多个节点的功能。包括：
  * 高级调度策略 


## 设计

Minikube 通过  [libmachine](https://github.com/docker/machine/tree/master/libmachine) 提供虚拟机，及使用 [localkube](https://git.k8s.io/minikube/pkg/localkube)（最初由 [RedSpread](https://redspread.com/)编写并捐赠给此项目）来运行集群。

有关 minikube 的更多信息，请参阅[提案](https://git.k8s.io/community/contributors/design-proposals/cluster-lifecycle/local-cluster-ux.md)。


## 其它链接

* **目标和非目标**：对于 minikube 项目的目标和非目标，请参阅我们的 [路线图](https://git.k8s.io/minikube/docs/contributors/roadmap.md)。
* **开发指南**：请参阅 [CONTRIBUTING.md](https://git.k8s.io/minikube/CONTRIBUTING.md) 了解如何发送 pull request。
* **构建 Minikube**：有关如何从源代码构建/测试 minikube 的说明，请参阅 [构建指南](https://git.k8s.io/minikube/docs/contributors/build_guide.md)。
* **添加一个新依赖**：有关如何添加一个新依赖到 minikube 的说明，请参阅 [添加依赖指南](https://git.k8s.io/minikube/docs/contributors/adding_a_dependency.md)。
* **添加一个新插件**：有关如何添加一个新插件 minikube 的指导，请查看 [添加插件指南](https://git.k8s.io/minikube/docs/contributors/adding_an_addon.md)。
* **更新 Kubernetes**：有关如何更新 Kubernetes 的说明，请参阅 [更新Kubernetes指南](https://git.k8s.io/minikube/docs/contributors/updating_kubernetes.md)。


## 社区

我们欢迎所有的贡献、问题和评论！Minikube 开发者在 [Slack](https://kubernetes.slack.com) 的 #minikube 频道中闲聊（从 [这里](http://slack.kubernetes.io/) 获得一个邀请）。我们也有 [kubernetes-dev Google Groups 邮件列表](https://groups.google.com/forum/#!forum/kubernetes-dev)。如果您正在向列表中发送，请在主题前加上 "minikube: "。
