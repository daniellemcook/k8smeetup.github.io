---
approvers:
- bryk
- mikedanese
- rf232
cn-approvers:
- zhangqx2010
title: Web UI (Dashboard)
---



Dashboard 是基于 web 的 Kubernetes 用户界面。使用 Dashboard，您可以将容器化应用部署到 Kubernetes 集群中，也可以对容器化应用排错，还能管理集群本身及其附属资源。您可以使用 Dashboard 获取运行在集群中的应用的概览信息，也可以创建或者修改 Kubernetes（如 Deployment，Job，DaemonSet 等等）资源。例如，您可以对 Deployment 实现弹性伸缩、创建滚动升级、重启 Pod 或者使用向导创建新的应用。

Dashboard 同时展示了 Kubernetes 集群中的资源状态信息和所有报错信息。

![Kubernetes Dashboard UI](/images/docs/ui-dashboard.png)

* TOC
{:toc}


## 部署 Dashboard UI

默认情况下，没有部署 Dashboard。如果想要部署它，请运行如下命令：

```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```


## 访问 Dashboard UI

访问 Dashboard UI 有很多方式；可以使用 kubectl 命令行接口，或者使用浏览器访问 Kubernetes master apiserver。


### 命令行代理
您可以使用 kubectl 命令行工具访问 Dashboard，命令如下：

```
kubectl proxy
```


Kubectl 会处理与 apiserver 的认证过程，并使得 Dashboard 能够通过 http://localhost:8001/ui 访问。

UI 只能通过执行这条命令的机器进行访问。更多选项参见 `kubectl proxy --help`。


### Master 服务器
UI 可以直接通过 Kubernetes master apiserver 访问。打开浏览器，输入 `https://<kubernetes-master>/ui`，其中 `<kubernetes-master>` 是 Kubernetes master 的 IP 地址或者域名。

请注意，只有当 apiserver 允许使用用户名密码认证时，这中方式才可以正常工作。但对于安装工具（如 `kubeadm`）来说并没有设置。关于如何手工设置认证，参见 [authentication 管理文档](/docs/admin/authentication/)。

如果您不知道配置的用户名密码，使用 `kubectl config view` 查询。


## 欢迎界面

当访问空集群的 Dashboard 时，您会看到欢迎界面。页面包含一个指向此文档的链接，以及一个用于部署第一个应用程序的按钮。此外，您可以看到在默认情况下有哪些系统应用运行在 `kube-system` [namespace](/docs/tasks/administer-cluster/namespaces/) 中，比如 Dashboard 自己。

![Kubernetes Dashboard welcome page](/images/docs/ui-dashboard-zerostate.png)


## 部署容器化应用

通过一个简单的向导，您可以使用 Dashboard 将容器化应用作为一个 Deployment 和可选的 Service 进行创建和部署。可以手工指定应用的详细配置，或者上传一个包含应用配置的 YAML 或 JSON 文件。

点击 respective 按钮访问欢迎界面上的部署向导。向导也可以通过点击任何页面右上角的 **CREATE** 按钮进行快速访问。

![Deploy wizard](/images/docs/ui-dashboard-deploy-simple.png)


### 指定应用的详细配置

部署向导需要您提供以下信息：


- **App name**（必填）：应用的名称。带有名称的 [label](/docs/concepts/overview/working-with-objects/labels/) 会被写入任何将被部署的 Deployment 和 Service。

  在选定的 Kuberntes [namespace](/docs/tasks/administer-cluster/namespaces/) 中，要求应用名称唯一。必须由小写字母开头，以数字或者小写字母结尾，并且只含有小写字母、数字和中划线（-）。小于等于24个字符。开头和结尾的空格会被忽略。


- **Container image**（必填）：公共镜像仓库上的 Docker [container image](/docs/concepts/containers/images/) 的 URL 或者私有镜像（通常是 Google Container Registery 或者 Docker Hub）。容器镜像参数必须以冒号结尾。


- **Number of pods**（必填）：您希望应用程序部署的 Pod 的数量。值必须为正整数。

  会创建一个 [Deployment](/docs/concepts/workloads/controllers/deployment/) 用于保证集群中运行了期望的 Pod 数量。


- **Service**（可选）：对于应用的某些部分（比如前端），您可能想对外暴露 [Service](/docs/concepts/services-networking/service/) ，可能是集群外部（external Service）的公网地址。对于外部服务，需要开放一个或者多个端口来满足。更多信息请参考 [这里](/docs/tasks/access-application-cluster/configure-cloud-provider-firewall/。

  其它只能对集群内部可见对 Service 称为 internal Service。

  不管哪种 Service 类型，如果您选择创建一个 Service，而且容器在一个端口上开启了监听（入向的），那么您需要定义两个端口。创建的 Service 会将（入向的）端口映射到容器可见的目标端口。Service 将会路由到您部署的 Pod。支持 TCP 和 UDP 协议。这个 Service 内部的 DNS 解析名就是之前您定义的应用名称。


如果需要，您可以打开 **高级选项** 部分，这里您可以定义更多设置：


- **Description**：这里您输入的文本会作为一个 [annotation](/docs/concepts/overview/working-with-objects/annotations/) 添加到 Deployment，并显示在应用的详细信息中。


- **Labels**：应用默认使用的 [labels](/docs/concepts/overview/working-with-objects/labels/) 是应用名称和版本。您可以为 Deployment、Service（如果有）定义额外的 label，比如 release、environment、tier、partition 和 release track。


  例子：

  ```conf
release=1.0
tier=frontend
environment=pod
track=stable
```


- **Namespace**：Kubernetes 支持多个虚拟集群依附于同一个物理集群。这些虚拟集群被称为 [namespaces](/docs/tasks/administer-cluster/namespaces/)。它们让您将资源划分为逻辑命名的组。

  Dashboard 通过下拉菜单提供所有可用的 namespace，并允许您创建新的 namespace。namespace 名称最长可以包含 63 个字母或者数字和中横线（-），但是不能包含大写字母。

  在 namespace 创建成功的情况下，默认会选中新的 namespace 。如果创建失败，那么第一个 namespace 会被选中。


- **Image Pull Secret**：想要使用私有的 Docker 容器镜像，需要 [pull secret](/docs/concepts/configuration/secret/) 凭证。

  Dashboard 通过下拉菜单提供所有可用的 secret，并允许您创建新的 secret。secret 名称必须遵循 DNS 域名语法，比如 `new.image-pull.secret`。secret 的内容必须是 base64 编码的，并且在一个 [`.dockercfg`](/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod) 文件中声明。secret 名称最大可以包含 253 个字符。

  在 image pull secret 创建成功的情况下，默认会选中新的 secret 。如果创建失败，则不会应用任何 secret。


- **CPU requirement (cores)** 和 **Memory requirement (MiB)**：您可以为容器定义最小的 [资源限制](/docs/tasks/configure-pod-container/limit-range/)。默认情况下，Pod 的运行没有 CPU 和内存限制。


- **Run command** 和 **Run command arguments**：默认情况下，您的容器会运行 Docker 镜像的默认 [入口命令](/docs/user-guide/containers/#containers-and-commands)。您可以使用 command 选项覆盖默认值。


- **Run as privileged**：这个设置决定了在 [特权容器](/docs/user-guide/pods/#privileged-mode-for-pod-containers) 中运行的进程是否像主机中使用 root 运行的进程一样。特权容器可以使用诸如操纵网络堆栈和访问设备的功能。


- **Environment variables**：Kubernetes 通过 [环境变量](/docs/tasks/inject-data-application/environment-variable-expose-pod-information/) 暴露服务。您可以构建环境变量，或者将环境变量的值作为参数传递给您的命令。应用可以使用他们查找服务。值可以通过  `$(VAR_NAME)` 语法关联其他变量。


### 上传 YAML 或者 JSON 文件

Kubernetes 支持声明式配置。所有的配置都存储在遵循 Kubernetes [API](/docs/concepts/overview/kubernetes-api/) 资源 schema 的 YAML 或者 JSON 配置文件中。

作为一种替代在部署向导中指定应用详情的方式，您可以在 YAML 或者 JSON 文件中定义应用，并且使用 Dashboard 上传文件：

![Deploy wizard file upload](/images/docs/ui-dashboard-deploy-file.png)


## 使用 Dashboard
以下各节描述了 Kubernetes Dashboard UI 视图；包括它们提供的内容，以及怎么使用它们。


### 导航栏

当在集群中定义 Kubernetes 对象时，Dashboard 会在初始视图中显示它们。默认情况下只会显示 _default_ namespace 中的对象，可以通过更改导航栏菜单中的 namespace 选择器进行改变。

Dashboard 展示大部分 Kubernetes 对象，并将它们分组放在几个菜单类别中。


#### 管理
集群和 namespace 管理的视图。它会列出 Node、Namespace 和 Persistent Volume，并且有它们的详细视图。Node 列表视图包含从所有 Node 聚合的 CPU 和内存使用的度量值。详细信息视图显示了一个 Node 的度量值，它的规格、状态、分配的资源、事件和这个节点上运行的 Pod。

![Node detail view](/images/docs/ui-dashboard-node.png)


#### 负载
入口（Entry point）视图显示选中 namespace 中所有运行的应用。视图按照负载类型（如 Deployment、Replica Set、Stateful Set 等）罗列应用，并且每种负载都可以单独查看。列表总结了关于负载的可执行信息，比如一个 Replica Set 的 ready 状态的 Pod 数量，或者目前一个 Pod 的内存使用量。

![Workloads view](/images/docs/ui-dashboard-workloadview.png)


负载的详细视图展示状态、详细信息和对象间的关系。例如，Replica Set 所控制的 Pod，或者 Deployments 关联的 New Replica Set 和 Horizontal Pod Autoscaler。

![Deployment detail view](/images/docs/ui-dashboard-deployment-detail.png)


#### 服务（Service）和发现（discovery）
服务和发现视图展示允许暴露给外网服务和允许集群内部发现的 Kubernetes 资源。因此，Service 和 Ingress 视图展示他们关联的 Pod、给集群连接使用的内部端点和给外部用户使用的外部端点。

![Service list partial view](/images/docs/ui-dashboard-service-list.png)


#### 存储
存储视图展示 PVC 资源（Persistent Volume Claim），这些资源被用于存储数据的应用程序使用。


#### 配置
配置视图展示的所有 Kubernetes 资源是在集群中运行的应用程序的实时配置。目前来说就是 Config Map 和 Secret。通过这个视图可以编辑和管理 config 对象，并显示那些默认隐藏的 secret。

![Secret detail view](/images/docs/ui-dashboard-secret-detail.png)


#### 日志查看器
Pod 列表和详细信息页面可以链接到 Dashboard 内置的日志查看器。查看器可以获取同一个 Pod 中容器的日志。

![Logs viewer](/images/docs/ui-dashboard-logs-view.png)


## 更多信息

参见 [Kubernetes Dashboard 项目页面](https://github.com/kubernetes/dashboard)。