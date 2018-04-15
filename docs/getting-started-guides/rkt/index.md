---
cn-approvers:
- tianshapjq
approvers:
- yifan-gu
title: 通过 rkt 运行 Kubernetes
---



本文档描述如何使用 [rkt](https://github.com/coreos/rkt) 作为容器运行时（runtime）来运行 Kubernetes。


*注意*：本文档描述如何使用所谓的 "rktnetes"。在以后，Kubernetes 将通过容器运行时接口（CRI）来支持 rkt 运行时（runtime）。目前 [rkt shim for the CRI](https://github.com/kubernetes-incubator/rktlet) 仍处于实验阶段，不过如果需要使用仍然可以在 [kubeadm reference](/docs/admin/kubeadm/#use-kubeadm-with-other-cri-runtimes) 中找到教程。

* TOC
{:toc}


## 前提


* [Systemd](http://www.freedesktop.org/wiki/Software/systemd/) 必须安装并且启用。Kubernetes V1.3 要求的 systemd 最低版本是219。Systemd 是用来监控和管理 node 上的 pod 的。


* [安装最新版的 rkt](https://coreos.com/rkt/docs/latest/trying-out-rkt.html)。要求的 rkt 最低版本是 [v1.13.0](https://github.com/coreos/rkt/releases/tag/v1.13.0)。[CoreOS Linux alpha channel](https://coreos.com/releases/) 附带了一个最新的 rkt 发布版本，如果必要的话您可以通过 [在 CoreOS 上更新 rkt](https://coreos.com/rkt/docs/latest/install-rkt-in-coreos.html) 更新 rkt。


* [rkt API service](https://coreos.com/rkt/docs/latest/subcommands/api-service.html) 必须运行在 node 上。


* 你需要安装 [kubelet](/docs/getting-started-guides/scratch/#kubelet) 在 node 上，并且建议运行 [kube-proxy](/docs/getting-started-guides/scratch/#kube-proxy) 在所有 node 上。本文档描述如何设置 kubelet 的参数使其能够使用 rkt 作为运行时（runtime）。


## rktnetes 中的 pod 网络


### Kubernetes 的 CNI 网络


你可以通过适当地设置 kubelet 的 `--network-plugin` 和 `--network-plugin-dir` 参数来让 Kubernetes pod 使用常用的 Container Network Interface（CNI）[网络插件](/docs/concepts/cluster-administration/network-plugins/)。通过这种方法，rkt 可以在不了解网络细节的情况下通过提供的子网来连接 pod。


#### kubenet：Google Compute Engine（GCE）网络


您能够通过配置 kubelet 的参数 `--network-plugin=kubenet` 来选择使用 `kubenet` 插件。该插件目前只支持GCE环境。当使用 kubenet，Kubernetes CNI 将会创建并且管理网络，同时提供一个网桥设备给 rkt 使其能够连接到 GCE 网络。


### rkt contained 网络


相对于将网络交给 Kubernetes，rkt 也能通过其自身的 [*contained 网络*](https://coreos.com/rkt/docs/latest/networking/overview.html#contained-mode) 在一个网桥设备提供的子网上直接配置网络连接，如 flannel SDN，或者其它的 CNI 插件。以这种方式配置，rkt 通过查找它的 [配置目录](https://coreos.com/rkt/docs/latest/configuration.html#command-line-flags)，通常位于 `/etc/rkt/net.d`，来发现 CNI 配置并且调用合适的插件来创建 pod 网络。


#### 通过网桥实现 rkt contained 网络


rkt 默认使用 *contained 网络*，所以你可以把 kubelet 的 `--network-plugin` 参数置空让其使用这个默认网络。任意的 CNI 插件都能支持 contained 网络。通过 *contained 网络*，rkt 将尝试把 pod 连接到一个名为 `rkt.kubernetes.io` 的网络，所以提供支持的 CNI 配置必须使用这个名称。


如果使用 contained 网络，需要在 rkt 网络配置目录下创建一个网络配置文件，该文件定义如何在您的环境中创建 `rkt.kubernetes.io` 网络。如下示例展示如何通过 `bridge` CNI 插件创建一个网桥设备：

```shell
$ cat <<EOF >/etc/rkt/net.d/k8s_network_example.conf
{
  "name": "rkt.kubernetes.io",
  "type": "bridge",
  "bridge": "mybridge",
  "mtu": 1460,
  "addIf": "true",
  "isGateway": true,
  "ipMasq": true,
  "ipam": {
    "type": "host-local",
    "subnet": "10.22.0.0/16",
    "gateway": "10.22.0.1",
    "routes": [
      { "dst": "0.0.0.0/0" }
    ]
  }
}
EOF
```


#### 通过 flannel 实现 rkt contained 网络


目前推荐通过 Kubernetes CNI 的支持来操作 flannel，不过您也可以通过配置 flannel 插件目录来为 rkt 的 contained 网络提供子网。如下提供一个 CNI/flannel 配置文件的示例：

```shell
$ cat <<EOF >/etc/rkt/net.d/k8s_flannel_example.conf
{
    "name": "rkt.kubernetes.io",
    "type": "flannel",
    "delegate": {
        "isDefaultGateway": true
    }
}
EOF
```


如果需要查看更多关于 flannel 的配置信息，请参阅 [CNI/flannel README](https://github.com/containernetworking/plugins/blob/master/plugins/meta/flannel/README.md)。


#### Contained 网络的注意事项：


* 对于创建的 CNI 配置文件，其中的网络名称必须为 `rkt.kubernetes.io`。
* 向下兼容的 API 和环境变量替换将不包含 pod IP 地址。
* 尽管提供了 `/etc/hostname`， 但是 `/etc/hosts` 文件将不会包含 pod 自己的 hostname。


## 运行 rktnetes


### 通过 rkt 运行时（runtime）来运行一个本地的 Kubernetes 集群


如果需要使用 rkt 来作为本地 Kubernetes 集群的容器运行时（container runtime），需要为 kubelet 设置如下参数：


* `--container-runtime=rkt` 设置 node 的容器运行时（container runtime）为 rkt。
* `--rkt-api-endpoint=HOST:PORT` 设置 rkt API service 的 endpoint。默认为：`localhost:15441`。
* `--rkt-path=PATH_TO_RKT_BINARY` 设置 rkt 二进制的目录。非必填项。如果不设置该参数，则在 `$PATH` 中查找 rkt。
* --rkt-stage1-image=STAGE1 设置 stage1 镜像的名字，例如 `coreos.com/rkt/stage1-coreos`。非必填项。如果不设置，则使用默认的 Linux 内核软件隔离器 stage1。


如果您是使用 [hack/local-up-cluster.sh](https://github.com/kubernetes/kubernetes/tree/{{page.githubbranch}}/hack/local-up-cluster.sh) 脚本来启动集群，您可以通过编辑 `CONTAINER_RUNTIME`，`RKT_PATH`，和 `RKT_STAGE1_IMAGE` 环境变量来设置上面这些参数。如果 `rkt` 已经正确的配置在您的 $PATH` 中，那么 `RKT_PATH` 和 `RKT_STAGE1_IMAGE` 就都不是必填项。

```shell
$ export CONTAINER_RUNTIME=rkt
$ export RKT_PATH=<rkt_binary_path>
$ export RKT_STAGE1_IMAGE=<stage1-name>
```


现在可以使用 `local-up-cluster.sh` 脚本来启动集群了：

```shell
$ hack/local-up-cluster.sh
```


同时，我们也尝试在 [minikube](https://github.com/kubernetes/minikube/issues/168) 中将 rkt 作为容器运行时（container runtime）。


### 在 Google Compute Engine（GCE）上启动 rktnetes 集群


本节概述如何使用 `kube-up` 脚本在 GCE 上启动 CoreOS/rkt 集群。


首先需要指定 OS distribution，GCE distributor 的 master project，和 Kubernetes 的 master 和 node 的示例镜像。然后设置 `KUBE_CONTAINER_RUNTIME` 为 `rkt`：

```shell
$ export KUBE_OS_DISTRIBUTION=coreos
$ export KUBE_GCE_MASTER_PROJECT=coreos-cloud
$ export KUBE_GCE_MASTER_IMAGE=<image_id>
$ export KUBE_GCE_NODE_PROJECT=coreos-cloud
$ export KUBE_GCE_NODE_IMAGE=<image_id>
$ export KUBE_CONTAINER_RUNTIME=rkt
```


或者，通过设置 `KUBE_RKT_VERSION` 来指定 rkt 版本：

```shell
$ export KUBE_RKT_VERSION=1.13.0
```


或者，通过设置 `KUBE_RKT_STAGE1_IMAGE` 来为容器运行时（container runtime）选择另一个 [stage1 isolator](#modular-isolation-with-interchangeable-stage1-images)

```shell
$ export KUBE_RKT_STAGE1_IMAGE=<stage1-name>
```


然后您就可以通过以下命令启动集群：

```shell
$ cluster/kube-up.sh
```


### 在 AWS 上启动一个 rktnetes 集群


在 AWS 上目前还不支持 `kube-up` 脚本。不过，我们建议遵循 [Kubernetes on AWS guide](https://coreos.com/kubernetes/docs/latest/kubernetes-on-aws.html) 在 AWS 上启动一个 CoreOS Kubernetes 集群，然后按照上述配置来设置 kubelet 的选项。


### 在集群中部署应用


创建集群之后，您就可以开始部署应用了。您可以在 [deploy a simple nginx web server](/docs/user-guide/simple-nginx) 找到一个介绍示例。请注意，您并不需要修改该示例来让其运行在 "rktnetes" 集群上。更多示例可以参阅 [Kubernetes examples directory](https://github.com/kubernetes/examples/tree/{{page.githubbranch}}/)。


## 通过可互换的 stage1 镜像实现模块化隔离


rkt 在一个可互换的隔离环境中运行容器。这个功能被称作 [*stage1* 镜像](https://coreos.com/rkt/docs/latest/devel/architecture.html#stage-1)。目前支持三种 rkt stage1 镜像：


* 默认使用 `systemd-nspawn` stage1。通过 Linux kernel namespaces 和 cgroups 来隔离运行态的容器，该行为类似于默认的容器运行时（container runtime）。
* [`KVM` stage1](https://coreos.com/rkt/docs/latest/running-lkvm-stage1.html)，在一个 KVM hypervisor-managed 的虚拟机中运行容器。在 Kubernetes v1.3 版本中仍处于实验状态。
* [`fly stage1`](https://coreos.com/rkt/docs/latest/running-fly-stage1.html)，仅通过 `chroot` 来隔离容器，从而为需要特殊权限的应用程序提供主机级别的 mount 和网络命名空间权限。


除了上述的三个 stage1 镜像，您也可以根据您自己特定的隔离需求来 [创建您自己的镜像]https://coreos.com/rkt/docs/latest/devel/stage1-implementors-guide.html)。如果不配置，那么将默认使用 [default stage1](https://coreos.com/rkt/docs/latest/build-configure.html#parameters-for-setting-up-default-stage1-image)。目前有两种方法来选择不同的 stage1；通过每个 node 或者每个 pod：


* 设置 kubelet 的 `--rkt-stage1-image` 参数来告知 kubelet 为其 node 上的所有 pod 使用哪个 stage1 镜像。例如，`--rkt-stage1-image=coreos/rkt/stage1-coreos` 将会选择默认的 systemd-nspawn stage1。
* 设置 `rkt.alpha.kubernetes.io/stage1-name-override` 注解来覆盖 stage1，stage1 用来执行一个指定的容器。这将允许在同一个集群或者节点上混合使用不同的容器隔离机制。例如，以下的 pod manifest （缩减版）将使用 `fly stage1` 来运行 pod，从而让它的应用 -- 这里就是 `kubelet` -- 能够访问宿主机的命名空间：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubelet
  namespace: kube-system
  labels:
    k8s-app: kubelet
  annotations:
    rkt.alpha.kubernetes.io/stage1-name-override: coreos.com/rkt/stage1-fly
spec:
  containers:
  - name: kubelet
    image: quay.io/coreos/hyperkube:v1.3.0-beta.2_coreos.0
    command:
    - kubelet
    - --api-servers=127.0.0.1:8080
    - --config=/etc/kubernetes/manifests
    - --allow-privileged
    - --kubeconfig=/etc/kubernetes/kubeconfig
    securityContext:
      privileged: true
[...]
```


### 使用不同 stage1 镜像的注意事项


设置 stage1 注解将会潜在的赋予 pod 以 root 权限。所以，pod 的 `securityContext` 中的 `privileged` 必须设置为 `true`。


使用 KVM stage1 时需要配套使用 rkt 的 [*contained network*](#rkt-contained-network)，因为 CNI 插件驱动目前还没完全支持 hypervisor-based 运行时（runtime）。


## rkt 和 Docker 之间已知的问题和差异


rkt 和默认节点容器引擎具有非常不同的设计，就像 rkt 的本地 ACI 和 Docker 的容器镜像格式一样。从一个容器引擎切换到另一个时，用户可能会遇到不同的行为。更多信息请参阅 [in the Kubernetes rkt notes](/docs/getting-started-guides/rkt/notes/)。


## 故障排除


以下是一些基于 rkt 容器引擎的 Kubernetes 的故障排除的提示：


### 检查 rkt 的 pod 状态


如果想要检查正在运行的 pod 的状态，可以使用 rkt 的子命令 [`rkt list`](https://coreos.com/rkt/docs/latest/subcommands/list.html)，[`rkt status`](https://coreos.com/rkt/docs/latest/subcommands/status.html)，和 [`rkt image list`](https://coreos.com/rkt/docs/latest/subcommands/image.html#rkt-image-list)。更多关于 rkt 子命令的信息可参阅 [rkt commands documentation](https://coreos.com/rkt/docs/latest/commands.html)。


### 检查 journal 日志


可以在 node 上使用 `journalctl` 命令来检查 pod 的日志。Pod 是作为 systemd 单元来进行管理和命名的。Pod 的单元名称是通过把 `k8s_` 前缀和 pod 的 UUID 串联起来形成的，格式类似于 `k8s_${RKT_UUID}`。用 `rkt list` 命令找到 pod 的 UUID 来组成它的服务名，然后使用 journalctl 命令查看日志：


```shell
$ sudo journalctl -u k8s_ad623346
```


#### 日志输出级别


默认情况下，日志级别为2。如果想要查看更多和 rkt 相关的日志信息，可以设置级别为4或更高级别。对于一个本地集群，设置环境变量：`LOG_LEVEL=4` 即可。


检查 Kubernetes 的事件和日志。


Kubernetes 提供了多种工具以进行问题排查和检测。更多信息可参阅 [in the app troubleshooting guide](/docs/tasks/debug-application-cluster/debug-application/)。
