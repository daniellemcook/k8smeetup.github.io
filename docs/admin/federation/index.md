---
approvers:
- madhusudancs
- mml
- nikhiljindal
title: （已废弃） 使用 `federation-up` 和 `deploy.sh`
cn-approvers:
- chentao1596
---



## 本文描述的安装联邦的机制已废弃。[`kubefed`](/docs/tasks/federation/set-up-cluster-federation-kubefed/) 是现在推荐的部署联邦的方式。


本指南描述了如何建立集群联邦来让我们控制多个 Kubernetes 集群。

* TOC
{:toc}


## 先决条件


本指南假设您已经有一个正常运行的 Kubernetes 集群。如果需要启动新集群，请参考 [入门指南](/docs/setup/) 获取有关集群启动的说明。


要使用本指南中的命令，您必须从 [使用二进制版本](/docs/getting-started-guides/binary_release/) 中下载 Kubernetes 的版本并且提取到一个目录中；本指南中的所有命令都是在那个解压得到的目录中执行的。

```shell
$ curl -L https://github.com/kubernetes/kubernetes/releases/download/v1.4.0/kubernetes.tar.gz | tar xvzf -
$ cd kubernetes
```


您还必须安装一个在本地（运行本指南中描述的命令的计算机）运行的 Docker。


## 设置联邦控制平面


设置联邦需要运行的联邦控制平面，它由 etcd，federation-apiserver（通过 hyperkube 二进制文件） 以及 federation-controller-manager（也是通过 hyperkube 二进制文件)）组成。您可以将这些二进制文件以 pod 的方式在现有的 Kubernetes 集群中运行。


注：这是一种新的创建 Kubernetes 集群联邦的机制。如果您想遵循旧的机制，请参阅本指南末尾的章节描述的 [以前的联邦创建机制](#以前的联邦创建机制)。


### 初始设置


创建目录，用于存储创建联邦的配置，将环境变量 `FEDERATION_OUTPUT_ROOT` 导出为该目录的路径。这可以是一个已经存在的目录，但我们强烈建议创建一个单独的目录，以便以后更容易清理。

```shell
$ export FEDERATION_OUTPUT_ROOT="${PWD}/_output/federation"
$ mkdir -p "${FEDERATION_OUTPUT_ROOT}"
```


初始化设置

```shell
$ federation/deploy/deploy.sh init
```


或者，您可以创建或者编辑 `${FEDERATION_OUTPUT_ROOT}/values.yaml` 文件，去自定义任何 [federation/federation/manifests/federation/values.yaml](https://github.com/madhusudancs/kubernetes-anywhere/blob/federation/federation/manifests/federation/values.yaml) 中的值。例如：

```yaml
apiserverRegistry: "gcr.io/myrepository"
apiserverVersion: "v1.5.0-alpha.0.1010+892a6d7af59c0b"
controllerManagerRegistry: "gcr.io/myrepository"
controllerManagerVersion: "v1.5.0-alpha.0.1010+892a6d7af59c0b"
```


假设您已经使用上面例子中给定的标签构建并且推送 `hyperkube` 镜像到仓库中。


## 获取镜像


要将联邦控制平面组件作为 pod 运行，首先需要所有组件的镜像。您可以使用官方发布的镜像，也可以自己从头构建它们。


### 使用官方发布的镜像


作为每个 Kubernetes 版本发布的一部分，官方发布镜像将推送到 `k8s.gcr.io` 仓库中。要使用这个仓库中的镜像，您可以将下面配置中的容器镜像字段指向该仓库中的镜像。`k8s.gcr.io/hyperkube` 包含了 federation-apiserver 和 federation-controller-manager 二进制文件，因此，您可以将这些组件相应的配置指定为这个 hyperkube 镜像。



### 从头构建和推送镜像


要构建二进制文件，请签出 [Kubernetes 仓库](https://github.com/kubernetes/kubernetes)，并在源代码的根目录下运行下面这些命令：


```shell
$ federation/develop/develop.sh build_binaries
```


要构建镜像并将其推送到仓库，请运行：

```shell
$ KUBE_REGISTRY="gcr.io/myrepository" federation/develop/develop.sh build_image
$ KUBE_REGISTRY="gcr.io/myrepository" federation/develop/develop.sh push
```


注：执行过程会覆盖文件 `${FEDERATION_OUTPUT_ROOT}/values.yaml` 中可能你已经设置过的某些值，包括 `apiserverRegistry`，`apiserverVersion`，`controllerManagerRegistry` 以及
`controllerManagerVersion`。因此，如果您从源代码构建镜像的话，不建议自定义 `${FEDERATION_OUTPUT_ROOT}/values.yaml` 中的这些值。


### 运行联邦控制平面


一旦有了镜像，您可以通过运行下面命令创建联邦控制平面：

```shell
$ federation/deploy/deploy.sh deploy_federation
```


它将在您现有的 Kubernetes 集群中用 [`Deployment`](/docs/concepts/workloads/controllers/deployment/) 管理的 pod 来启动联邦控制平面组件。它还会为 `federation-apiserver` 启动一个 [`type: LoadBalancer`](/docs/concepts/services-networking/service/#type-loadbalancer) 类型的 [`Service`](/docs/concepts/services-networking/service/)，以及为 `etcd` 创建一个通过动态 [`PV`](/docs/concepts/storage/persistent-volumes/) 支持的 [`PVC`](/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims/) 。所有这些组件都创建在 `federation` 命名空间中。


您可以通过运行以下命令来验证 pod 是否可用：


```shell
$ kubectl get deployments --namespace=federation
NAME                            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
federation-apiserver            1         1         1            1           1m
federation-controller-manager   1         1         1            1           1m
```


运行 `deploy.sh` 也会在您的 kubeconfig 中创建一个新的记录，用于跟联邦 apiserver 通信。您可以运行 `kubectl config view` 进行查看。



注：目前，持久性卷的动态配置只适用于AWS、GoogleKubernetes 引擎和 GCE。但是，如果需要，您可以编辑创建的 `Deployment` 以满足您的需要。


## 向联邦注册 Kubernetes 集群


现在，联邦控制平面已经启动并运行，您可以开始注册 Kubernetes 集群了。


首先，您需要为该 Kubernetes 集群创建一个包含 kubeconfig 的 secret，联邦控制平面将使用该 secret 来与 Kubernetes 集群通信。现在，您可以在主机 Kubernetes 集群（托管联邦控制平面的集群）中创建这个 secret。当联邦开始支持 secret 时，您将能够在那里直接创建这个 secret。假设 Kubernetes 集群的 kubeconfig 位于 `/cluster1/kubeconfig`，您可以运行以下命令来创建 secret：

```shell
$ kubectl create secret generic cluster1 --namespace=federation --from-file=/cluster1/kubeconfig
```


注意，文件名应该是 `kubeconfig` ，因为文件名决定了 secret 中密钥的名称。


创建 secret 之后，就可以注册集群了。用于注册集群的 YAML 文件如下所示：

```yaml
apiVersion: federation/v1beta1
kind: Cluster
metadata:
  name: cluster1
spec:
  serverAddressByClientCIDRs:
  - clientCIDR: <client-cidr>
    serverAddress: <apiserver-address>
  secretRef:
    name: <secret-name>
```


您需要为 `<client-cidr>`，`<apiserver-address>` 和 `<secret-name>` 填入适当的值。这里的 `<secret-name>`  就是您刚才创建的 secret 的名称。serverAddressByClientCIDRs 包含客户端按照 CIDR 使用的各种服务地址。您可以将公网 IP 按照 CIDR 设置为 `"0.0.0.0/0"`，这样所有的客户端都能匹配。另外，如果想要内部客户端使用服务的 clusterIP，那么可以将它设置为 serverAddress。在这种情况下，客户端 CIDR 将是只匹配运行在该集群中的 pod 的 IP 的那些 CIDR。


假设您的 YAML 文件位于 `/cluster1/cluster.yaml`，可以运行以下命令来注册此集群：


```shell
$ kubectl create -f /cluster1/cluster.yaml --context=federation-cluster

```


通过指定 `--context=federation-cluster`，您将请求发送到联邦 apiserver。通过运行以下命令，可以确定集群注册是否成功：

```shell
$ kubectl get clusters --context=federation-cluster
NAME       STATUS    VERSION   AGE
cluster1   Ready               3m
```


## 更新 KubeDNS


向联邦注册集群后，您需要更新 KubeDNS，以便您的集群可以路由联邦服务请求。更新方法根据您的 Kubernetes 版本而有所不同：在 Kubernetes 1.5 或更高版本上，您必须通过 kube-dns 配置映射 将 `--federations` 标志传递到 kube-dns 中；在 1.4 或者更早的版本，则必须直接在其他集群的 kube-dns-rc 上设置 `--federations` 标志。


### Kubernetes 1.5+：通过配置映射（config map）传递 federations 标志到 kube-dns 中


对于 1.5+ 版本的 Kubernetes 集群，可以通过 kube-dns 配置映射将 `--federations` 标志传递到 kube-dns 中。标志使用以下格式：

```
--federations=${FEDERATION_NAME}=${DNS_DOMAIN_NAME}
```


若要将此标志传递给 KubeDNS，请在命名空间 `kube-system` 下创建名称为 `kube-dns` 的 config-map。config-map 应该如下所示：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-dns
  namespace: kube-system
data:
  federations: <federation-name>=<federation-domain-name>
```


这里的 `<federation-name>` 应该使用您想要给您的联邦取的名字代替，`federation-domain-name` 则应该被您想要在联邦 DNS 中使用的域名代替。


您可以在 [配置映射](/docs/tasks/configure-pod-container/configmap/) 找到有关配置映射的更多详细信息。


### Kubernetes 1.4 及更早版本：在 kube-dns-rc 上设置联邦标志


如果您的集群运行的是 Kubernetes 1.4 或更早的版本，则必须重新启动 KubeDNS，并传递给它一个 `--federations` 标志，该标志告诉它有效的联邦 DNS 主机名。标志使用以下格式：

```
--federations=${FEDERATION_NAME}=${DNS_DOMAIN_NAME}
```


若要使用 `--federations` 标志更新 KubeDNS，可以编辑现有的 kubedns replication controller（rc 副本控制器），在 pod 模板的 spec 中包含它，然后删除现有的 pod。之后，replication controller 就会使用更新后的模板重新创建 pod。


若要查找现有 kubedns replication controller 的名称，请运行以下命令：

```shell
$ kubectl get rc --namespace=kube-system
```


您应该会看到集群上所有 replication controller 的列表。kube-dns replication controller 应该有一个类似于 `kube-dns-v18` 一样的名称。若要编辑 replication controller，请按名称指定它，如下所示：

```shell
$ kubectl edit rc <rc-name> --namespace=kube-system
```

在 kube-dns replication controller 的最终 YAML 文件中，将 `--federations` 标志作为参数添加到 kube-dns 容器中。


然后，您必须删除现有的 kube-dns pod。可以先通过运行下面命令找到 pod：

```shell
$ kubectl get pods --namespace=kube-system
```


然后通过运行以下命令删除该 pod：

```shell
$ kubectl delete pods <pod-name> --namespace=kube-system
```


一旦完成了 kube-dns 配置，您的联邦就可以使用了。


## 卸载


为了将联邦控制平面卸载，请运行以下命令：

```shell
$ federation/deploy/deploy.sh destroy_federation
```


## 以前的联邦创建机制


这里描述了以前创建 Kubernetes 集群联邦的机制。建议使用新的创建机制。如果您愿意想用这个机制代替新的，请在 [https://github.com/kubernetes/kubernetes/issues/new](https://github.com/kubernetes/kubernetes/issues/new) 提一个 issue 告诉我们为什么新的机制在您的使用中不起作用。


### 获取镜像


要将它们作为 pod 运行，首先需要所有组件的镜像。你可以使用官方发布的镜像，也可以从头构建。


#### 使用官方发布的镜像


作为每个 Kubernetes 版本发布的一部分，官方发布镜像将推送到 `k8s.gcr.io` 仓库中。要使用这些镜像，请设置环境变量 `FEDERATION_PUSH_REPO_BASE=k8s.gcr.io`，它将一直使用最新的镜像。若要使用来自特定发行版的 hyperkube 镜像（包含 federation-apiserver 和
federation-controller-manager），请设置环境变量 `FEDERATION_IMAGE_TAG`。


#### 从头构建和推送镜像


要从头运行代码来构建和推送自己的镜像。您可以使用以下命令构建镜像：

```shell
$ FEDERATION=true KUBE_RELEASE_RUN_TESTS=n make quick-release
```


接下来，您需要将这些镜像推送到注册表中，如 Google Container Registry 或者 Docker Hub，这样集群就可以拉取它们。如果 Kubernetes 集群运行在 Google Compute Engine（GCE）上，那么您可以将镜像推送到 `gcr.io/<gce-project-name>`。推送镜像的命令如下所示：

```shell
$ FEDERATION=true FEDERATION_PUSH_REPO_BASE=gcr.io/<gce-project-name> ./build/push-federation-images.sh
```


### 运行联邦控制平面


一旦有了镜像，你就可以在现有的集群上以 pod 的方式运行。在现有 GCE 集群上运行这些 pod 的命令如下所示：

```shell
$ KUBERNETES_PROVIDER=gce FEDERATION_DNS_PROVIDER=google-clouddns FEDERATION_NAME=myfederation DNS_ZONE_NAME=myfederation.example FEDERATION_PUSH_REPO_BASE=k8s.gcr.io ./federation/cluster/federation-up.sh
```


`KUBERNETES_PROVIDER` 是云服务提供商。


`FEDERATION_DNS_PROVIDER` 可以是 `google-clouddns` 或者 `aws-route53`。如果它没有值，需要使用适当的值对它进行设置。`KUBERNETES_PROVIDER` 是 `gce`，`gke` 和 `aws` 当中的一个，它用于解析联邦服务的 dns 请求。服务控制器在底层 Kubernetes 集群中更新 service/pod 时，为提供者保存 DNS 记录。


`FEDERATION_NAME` 是一个您可以为联邦选择的名字。这是 DNS 路由中将出现的名称。


‘DNS_zone_NAME’ 是用于 DNS 记录的域。您需要购买这样一个域，然后对其进行配置，以便按照 `FEDERATION_DNS_PROVIDER` 将该域的 DNS 查询路由到适当的提供商。


运行这些命令将创建一个命名空间 `federation`，并创建两个 deployment：`federation-apiserver` 和 `federation-controller-manager`。您可以通过运行以下命令来验证 pod 是否可用：

```shell
$ kubectl get deployments --namespace=federation
NAME                            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
federation-apiserver            1         1         1            1           1m
federation-controller-manager   1         1         1            1           1m
```


运行 `federation-up.sh` 也会在您的 kubeconfig 中创建一个新的记录，用于跟联邦 apiserver 通信。您可以运行 `kubectl config view` 进行查看。


注：`federation-up.sh` 创建一个 federation-apiserver pod，它包含一个由持久卷（PV persistent volume）支持的 etcd 容器，用来持久化数据。目前，它仅能在 AWS，Google Kubernetes Engine 以及 GCE 上工作。如果需要的话，您可以编辑 `federation/manifests/federation-apiserver-deployment.yaml`，以满足您的需要。



## 更多信息

 * [联邦提议](https://git.k8s.io/community/contributors/design-proposals/multicluster/federation.md) 推动这项工作的详细用例。
