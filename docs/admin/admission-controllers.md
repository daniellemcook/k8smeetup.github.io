---
assignees:
- bprashanth
- davidopp
- derekwaynecarr
- erictune
- janetkuo
- thockin
cn-approvers:
- linyouchong
title: 使用准入控制插件
---


* TOC
{:toc}


## 什么是准入控制插件？


一个准入控制插件是一段代码，它会在请求通过认证和授权之后、对象被持久化之前拦截到达 API server 的请求。插件代码运行在 API server 进程中，必须将其编译为二进制文件，以便在此时使用。


在每个请求被集群接受之前，准入控制插件依次执行。如果插件序列中任何一个拒绝了该请求，则整个请求将立即被拒绝并且返回一个错误给终端用户。


准入控制插件可能会在某些情况下改变传入的对象，从而应用系统配置的默认值。另外，作为请求处理的一部分，准入控制插件可能会对相关的资源进行变更，以实现类似增加配额使用量这样的功能。


## 为什么需要准入控制插件？


Kubernetes 的许多高级功能都要求启用一个准入控制插件，以便正确地支持该特性。因此，一个没有正确配置准入控制插件的 Kubernetes API server 是不完整的，它不会支持您所期望的所有特性。


## 如何启用一个准入控制插件？


Kubernetes API server 支持一个标志参数 `admission-control` ，它指定了一个用于在集群修改对象之前调用的以逗号分隔的准入控制插件顺序列表。


## 每个插件的功能是什么？

### AlwaysAdmit


使用这个插件自行通过所有的请求。

### AlwaysPullImages


这个插件修改每一个新创建的 Pod 的镜像拉取策略为 Always 。这在多租户集群中是有用的，这样用户就可以放心，他们的私有镜像只能被那些有凭证的人使用。没有这个插件，一旦镜像被拉取到节点上，任何用户的 pod 都可以通过已了解到的镜像的名称（假设 pod 被调度到正确的节点上）来使用它，而不需要对镜像进行任何授权检查。当启用这个插件时，总是在启动容器之前拉取镜像，这意味着需要有效的凭证。

### AlwaysDeny


拒绝所有的请求。用于测试。


### DenyExecOnPrivileged （已废弃）


如果一个 pod 拥有一个特权容器，这个插件将拦截所有在该 pod 中执行 exec 命令的请求。


如果集群支持特权容器，并且希望限制最终用户在这些容器中执行 exec 命令的能力，我们强烈建议启用这个插件。


此功能已合并到 [DenyEscalatingExec](#denyescalatingexec)。

### DenyEscalatingExec


这个插件将拒绝在拥有衍生特权而具备访问宿主机能力的 pod 中执行 exec 和 attach 命令。这包括在特权模式运行的 pod ，可以访问主机 IPC 名称空间的 pod ，和访问主机 PID 命名空间的 pod 。


如果集群支持使用以衍生特权运行的容器，并且希望限制最终用户在这些容器中执行 exec 命令的能力，我们强烈建议启用这个插件。

### ImagePolicyWebhook


ImagePolicyWebhook 插件允许使用一个后端的 webhook 做出准入决策。您可以按照如下配置 admission-control 选项来启用这个插件:

```shell
--admission-control=ImagePolicyWebhook
```


#### 配置文件格式

ImagePolicyWebhook 插件使用了admission config 文件 `--admission-control-config-file` 来为后端行为设置配置选项。该文件可以是 json 或 yaml ，并具有以下格式:

```javascript
{
  "imagePolicy": {
     "kubeConfigFile": "path/to/kubeconfig/for/backend",
     "allowTTL": 50,           // time in s to cache approval
     "denyTTL": 50,            // time in s to cache denial
     "retryBackoff": 500,      // time in ms to wait between retries
     "defaultAllow": true      // determines behavior if the webhook backend fails
  }
}
```


这个配置文件必须引用一个 [kubeconfig](/docs/concepts/cluster-administration/authenticate-across-clusters-kubeconfig/) 格式的文件，并在其中配置指向后端的连接。且需要在 TLS 上与后端进行通信。


kubeconfig 文件的 cluster 字段需要指向远端服务，user 字段需要包含已返回的授权者。

```yaml
# clusters refers to the remote service.
clusters:
- name: name-of-remote-imagepolicy-service
  cluster:
    certificate-authority: /path/to/ca.pem    # CA for verifying the remote service.
    server: https://images.example.com/policy # URL of remote service to query. Must use 'https'.

# users refers to the API server's webhook configuration.
users:
- name: name-of-api-server
  user:
    client-certificate: /path/to/cert.pem # cert for the webhook plugin to use
    client-key: /path/to/key.pem          # key matching the cert
```

对于更多的 HTTP 配置，请参阅 [kubeconfig](/docs/concepts/cluster-administration/authenticate-across-clusters-kubeconfig/) 文档。


#### 请求载荷


当面对一个准入决策时，API server 发送一个描述操作的 JSON 序列化的 api.imagepolicy.v1alpha1.ImageReview 对象。该对象包含描述被审核容器的字段，以及所有匹配 `*.image-policy.k8s.io/*` 的 pod 注释。


注意，webhook API 对象与其他 Kubernetes API 对象一样受制于相同的版本控制兼容性规则。实现者应该知道对 alpha 对象的更宽松的兼容性，并检查请求的 "apiVersion" 字段，以确保正确的反序列化。此外，API server 必须启用 imagepolicy.k8s.io/v1alpha1 API 扩展组 (`--runtime-config=imagepolicy.k8s.io/v1alpha1=true`)。


请求载荷例子：

```
{  
  "apiVersion":"imagepolicy.k8s.io/v1alpha1",
  "kind":"ImageReview",
  "spec":{  
    "containers":[  
      {  
        "image":"myrepo/myimage:v1"
      },
      {  
        "image":"myrepo/myimage@sha256:beb6bd6a68f114c1dc2ea4b28db81bdf91de202a9014972bec5e4d9171d90ed"
      }
    ],
    "annotations":[  
      "mycluster.image-policy.k8s.io/ticket-1234": "break-glass"
    ],
    "namespace":"mynamespace"
  }
}
```


远程服务将填充请求的 ImageReviewStatus 字段，并返回允许或不允许访问。响应主体的 "spec" 字段会被忽略，并且可以省略。一个允许访问应答会返回：

```
{
  "apiVersion": "imagepolicy.k8s.io/v1alpha1",
  "kind": "ImageReview",
  "status": {
    "allowed": true
  }
}
```


不允许访问，服务将返回：

```
{
  "apiVersion": "imagepolicy.k8s.io/v1alpha1",
  "kind": "ImageReview",
  "status": {
    "allowed": false,
    "reason": "image currently blacklisted"
  }
}
```


更多的文档，请参阅 `imagepolicy.v1alpha1` API 对象 和 `plugin/pkg/admission/imagepolicy/admission.go`。


使用注解进行扩展


一个 pod 中匹配 `*.image-policy.k8s.io/*` 的注解都会被发送给 webhook。这允许了解镜像策略后端的用户向它发送额外的信息，并为不同的后端实现接收不同的信息。


您可以在这里输入的信息有：


 * 在紧急情况下，请求 "break glass" 覆盖一个策略。

 * 从一个记录了 break-glass 的请求的票证系统得到的一个票证编号

 * 向策略服务器提供一个提示，用于提供镜像的 imageID，以方便它进行查找


在任何情况下，注解都是由用户提供的，并不会被 Kubernetes 以任何方式进行验证。在将来，如果一个注解确定将被广泛使用，它可能会被提升为  ImageReviewSpec 的一个命名字段。

### ServiceAccount


这个插件实现了 [serviceAccounts](/docs/user-guide/service-accounts) 的自动化。
如果您打算使用 Kubernetes 的 ServiceAccount 对象，我们强烈建议您使用这个插件。

### SecurityContextDeny


该插件将拒绝任何试图设置特定扩展 [SecurityContext](/docs/user-guide/security-context) 字段的 pod。如果集群没有使用 [ pod 安全策略](/docs/user-guide/pod-security-policy)  来限制安全上下文所能获取的值集，那么应该启用这个功能。

### ResourceQuota


此插件将观察传入的请求，并确保它不违反任何一个 `Namespace` 中的 `ResourceQuota` 对象中枚举出来的约束。如果您在 Kubernetes 部署中使用了 `ResourceQuota`
，您必须使用这个插件来强制执行配额限制。


请查看 [resourceQuota 设计文档](https://git.k8s.io/community/contributors/design-proposals/admission_control_resource_quota.md) 和 [Resource Quota 例子](/docs/concepts/policy/resource-quotas/) 了解更多细节。


强烈建议将这个插件配置在准入控制插件序列的末尾。这样配额就不会过早地增加，只会在稍后的准入控制中被拒绝。

### LimitRanger


这个插件将观察传入的请求，并确保它不会违反 `Namespace` 中 `LimitRange` 对象枚举的任何约束。如果您在 Kubernetes 部署中使用了 `LimitRange` 对象，则必须使用此插件来执行这些约束。LimitRanger 插件还可以用于将默认资源请求应用到没有指定任何内容的 Pod ；当前，默认的 LimitRanger 对 `default` 名称空间中的所有 pod 应用了0.1 CPU 的需求。


请查看 [limitRange 设计文档](https://git.k8s.io/community/contributors/design-proposals/admission_control_limit_range.md) 和 [Limit Range 例子](/docs/tasks/configure-pod-container/limit-range/) 了解更多细节。


### InitialResources （试验）


此插件观察 pod 创建请求。如果容器忽略了 requests 和 limits 计算资源，那么插件就会根据运行相同镜像的容器的历史使用记录来自动填充计算资源请求。如果没有足够的数据进行决策，则请求将保持不变。当插件设置了一个计算资源请求时，它会用它自动填充的计算资源对 pod 进行注解。


请查看 [InitialResouces 建议书](https://git.k8s.io/community/contributors/design-proposals/initial-resources.md) 了解更多细节。

### NamespaceLifecycle


这个插件强制不能在一个正在被终止的 `Namespace` 中创建新对象，和确保使用不存在 `Namespace` 的请求被拒绝。


删除 `Namespace` 触发了在该名称空间中删除所有对象（ pod 、 services 等）的一系列操作。为了确保这个过程的完整性，我们强烈建议启用这个插件。

### DefaultStorageClass


这个插件观察不指定 storage class 字段的 `PersistentVolumeClaim` 对象的创建，并自动向它们添加默认的 storage class 。这样，不指定 storage class 字段的用户根本无需关心它们，它们将得到默认的 storage class 。


当没有配置默认 storage class 时，这个插件不会执行任何操作。当一个以上的 storage class 被标记为默认时，它拒绝 `PersistentVolumeClaim` 创建并返回一个错误，管理员必须重新检查 `StorageClass` 对象，并且只标记一个作为默认值。这个插件忽略了任何 `PersistentVolumeClaim` 更新，它只对创建起作用。


查看 [persistent volume](/docs/user-guide/persistent-volumes) 文档了解 persistent volume claims 和 storage classes 并了解如何将一个 storage classes 标志为默认。

### DefaultTolerationSeconds


这个插件设置了 pod 默认的宽恕容忍时间，对于那些没有设置宽恕容忍时间的 pod ，可以容忍 `notready:NoExecute` 和 `unreachable:NoExecute` 这些 taint 5分钟。

### PodNodeSelector


通过读取命名空间注释和全局配置,这个插件默认并限制了在一个名称空间中使用什么节点选择器。


#### 配置文件格式

PodNodeSelector 插件使用准入配置文件 `--admission-control-config-file` 来设置后端行为的配置选项。


请注意，配置文件格式将在未来版本中移至版本化文件。


这个文件可能是 json 或 yaml ，格式如下：

```yaml
podNodeSelectorPluginConfig:
 clusterDefaultNodeSelector: <node-selectors-labels>
 namespace1: <node-selectors-labels>
 namespace2: <node-selectors-labels>
```


#### 配置注解格式

PodNodeSelector 插件使用键为 `scheduler.alpha.kubernetes.io/node-selector` 的注解将节点选择器分配给 namespace 。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  annotations:
    scheduler.alpha.kubernetes.io/node-selector: <node-selectors-labels>
  name: namespace3
```

### PodSecurityPolicy


此插件负责在创建和修改 pod 时根据请求的安全上下文和可用的 pod 安全策略确定是否应该通过 pod。


对于 Kubernetes < 1.6.0 的版本，API Server 必须启用 extensions/v1beta1/podsecuritypolicy API 扩展组 (`--runtime-config=extensions/v1beta1/podsecuritypolicy=true`)。


查看 [Pod 安全策略文档](/docs/concepts/policy/pod-security-policy/) 了解更多细节。

### NodeRestriction


这个插件限制了 kubelet 可以修改的 `Node` 和 `Pod` 对象。 为了受到这个入场插件的限制，kubelet 必须在 `system：nodes` 组中使用凭证，并使用 `system：node：<nodeName>` 形式的用户名。这样的 kubelet 只允许修改自己的 `Node` API 对象，只能修改绑定到节点本身的 `Pod` 对象。

未来的版本可能会添加额外的限制，以确保 kubelet 具有正确操作所需的最小权限集。


## 是否有推荐的一组插件可供使用？


有。
对于 Kubernetes >= 1.6.0 版本，我们强烈建议运行以下一系列准入控制插件（顺序也很重要）

```shell
--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,ResourceQuota,DefaultTolerationSeconds
```


对于 Kubernetes >= 1.4.0 版本，我们强烈建议运行以下一系列准入控制插件（顺序也很重要）

```shell
--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota
```


对于 Kubernetes >= 1.2.0 版本，我们强烈建议运行以下一系列准入控制插件（顺序也很重要）

```shell
--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota
```


对于 Kubernetes >= 1.0.0 版本，我们强烈建议运行以下一系列准入控制插件（顺序也很重要）

```shell
--admission-control=NamespaceLifecycle,LimitRanger,SecurityContextDeny,ServiceAccount,PersistentVolumeLabel,ResourceQuota
```
