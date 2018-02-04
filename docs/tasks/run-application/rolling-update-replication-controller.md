---
approvers:
- janetkuo
cn-approvers:
- pigletfly
title: 使用 Replication Controller 进行滚动升级
---


* TOC
{:toc}


## 概览


**注意**: 创建应用副本的首选方法是使用 [Deployment](/docs/api-reference/{{page.version}}/#deployment-v1beta1-apps)，Deployment 反过来使用 [ReplicaSet](/docs/api-reference/{{page.version}}/#replicaset-v1beta1-extensions)。想要获取更多信息，请查看 [使用 Deployment 运行无状态应用程序](/docs/tasks/run-application/run-stateless-application-deployment/)。


为了不中断地更新服务，`kubectl` 支持所谓的 [滚动升级](/docs/user-guide/kubectl/{{page.version}}/#rolling-update)，滚动升级一次只更新一个 pod，而且不是同时停掉整个服务。查看 [滚动升级设计文档](https://git.k8s.io/community/contributors/design-proposals/cli/simple-rolling-update.md) 和 [滚动升级示例](/docs/tasks/run-application/rolling-update-replication-controller/) 获取更多信息。



需要注意的是，`kubectl rolling-update` 只支持 Replication Controller 。但是，如果您在使用 Replication Controller 部署应用程序，可以考虑将它们切换到 [Deployment](/docs/concepts/workloads/controllers/deployment/)。Deployment 是可以实现应用程序自动滚动升级的更高级别的声明式控制器，所以推荐使用 Deployment 。如果您想继续使用 Replication Controller 和 `kubectl rolling-update`，请继续阅读：


滚动升级会对 replication controller 管理的 pod 的配置应用变更。变更可以作为 replication
controller 的配置文件传递；或者，如果只是更新了镜像，可以直接指定新的容器镜像。


滚动升级通过下面步骤进行工作：

1. 使用更新后的配置创建一个新的 replication controller。
2. 增加新的 replication controller 的 replica 数量，减少老的 replication controller 的 replica 数量，直到达到正确的 replica 数量。
3. 删除原来的 replication controller。

滚动升级是通过 `kubectl rolling-update` 命令来初始化的：

    $ kubectl rolling-update NAME \
        ([NEW_NAME] --image=IMAGE | -f FILE)



## 传递配置文件

要通过使用配置文件初始化滚动升级，需要传递新文件给 `kubectl rolling-update` 命令：


    $ kubectl rolling-update NAME -f FILE
  
这个配置文件必须是：

* 指定了不同的 `metadata.name` 的值。
* 在 `spec.selector` 字段里至少重写了一个常见 label。
* 使用相同的 `metadata.namespace`。

在 [创建 Replication Controllers](/docs/tutorials/stateless-application/run-stateless-ap-replication-controller/) 文档里描述了 Replication controller 的配置文件。


### 示例

    // Update pods of frontend-v1 using new replication controller data in frontend-v2.json.
    $ kubectl rolling-update frontend-v1 -f frontend-v2.json

    // Update pods of frontend-v1 using JSON data passed into stdin.
    $ cat frontend-v2.json | kubectl rolling-update frontend-v1 -f -


## 更新容器镜像

如果只是为了更新容器镜像，需要传递新的镜像名和新的 controller 名称，镜像名要使用 `--image` 参数来标记：

    $ kubectl rolling-update NAME [NEW_NAME] --image=IMAGE:TAG

`--image` 参数值只支持单个容器的 pod。在多容器的 pod 上使用 `--image` 指定镜像名会返回错误。

如果没有指定 `NEW_NAME` ，新的 replication controller 会使用临时的名称来创建。一旦 rollout 完成，老的 controller 会被删除，并且新的 controller 会更新为原来的的名称。

如果 `IMAGE:TAG` 的值和当前的值是完全一样的，更新会失败。出于这个原因，我们建议使用特定版本的镜像 tag，而不是使用例如 `:latest` 这样的 tag 。从 `image:latest` 镜像到新的 `image:latest` 镜像进行滚动升级会失败，即使
`image:latest` tag 里的镜像已经更改过。而且，使用 `:latest` 是不推荐的，查看 [配置最佳实践](/docs/concepts/configuration/overview/#container-images) 获取更多信息。


### 示例


    // Update the pods of frontend-v1 to frontend-v2
    $ kubectl rolling-update frontend-v1 frontend-v2 --image=image:v2

    // Update the pods of frontend, keeping the replication controller name
    $ kubectl rolling-update frontend --image=image:v2


## 必选和可选字段

必选的字段有：

* `NAME`: 要更新的 replication controller 的名称

还有：

* `-f FILE`: 一个 replication controller 的配置文件，无论是 JSON 或者 YAML 文件格式。这个配置文件必须指定一个新的顶级的 `id` 值，并且至少包含了一个已经存在的 `spec.selector` 键值对。
查看 [运行无状态应用 Replication Controller](/docs/tutorials/stateless-application/run-stateless-ap-replication-controller/#replication-controller-configuration-file) 页面获取详细信息。
<br>
<br>
    或者:
<br>
<br>
* `--image IMAGE:TAG`: 要更新的镜像的名称和 tag。必须和当前指定的 image:tag 不同。

可选字段有：

* `NEW_NAME`: 只有和 `--image` 参数配合一起使用(不和 `-f FILE` 一起使用)。分配给新的 replication controller 的名称。
* `--poll-interval DURATION`: 滚动升级后轮询 controller 状态的间隔时间。有效的单位有 `ns` (纳秒)、`us` 或者 `µs` (微秒)、`ms` (毫秒)、 `s` (秒) 、`m` (分钟) 或者 `h` (小时)。单位可以组合使用，比如 `1m30s` 。默认值是 `3s` 。
* `--timeout DURATION`: controller 更新 pod 时等待其退出的最大时间。默认值是 `5m0s` 。有效的单位在上面的 `--poll-interval` 中描述过。
* `--update-period DURATION`: 更新 pods 的等待间隔时间。默认值是 `1m0s` 。有效的单位在上面的 `--poll-interval` 中描述过。

更多的有关 `kubectl rolling-update` 命令的信息可以查看 [`kubectl` reference](/docs/user-guide/kubectl/{{page.version}}/#rolling-update)。



## 演练

假设您运行了 1.7.9 版本的 nginx：

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```


要更新到 1.9.1 版本，您可以使用 [`kubectl rolling-update --image`](https://git.k8s.io/community/contributors/design-proposals/cli/simple-rolling-update.md) 指定新的镜像：

```shell
$ kubectl rolling-update my-nginx --image=nginx:1.9.1
Created my-nginx-ccba8fbd8cc8160970f63f9a2696fc46
```

在另外一个窗口，您可以看到 `kubectl` 在这些 pod 上 添加了一个 `deployment` label，label 的值是配置文件的哈希，为了区分新的和老的 pod。

```shell
$ kubectl get pods -l app=nginx -L deployment
NAME                                              READY     STATUS    RESTARTS   AGE       DEPLOYMENT
my-nginx-ccba8fbd8cc8160970f63f9a2696fc46-k156z   1/1       Running   0          1m        ccba8fbd8cc8160970f63f9a2696fc46
my-nginx-ccba8fbd8cc8160970f63f9a2696fc46-v95yh   1/1       Running   0          35s       ccba8fbd8cc8160970f63f9a2696fc46
my-nginx-divi2                                    1/1       Running   0          2h        2d1d7a8f682934a254002b56404b813e
my-nginx-o0ef1                                    1/1       Running   0          2h        2d1d7a8f682934a254002b56404b813e
my-nginx-q6all                                    1/1       Running   0          8m        2d1d7a8f682934a254002b56404b813e
```


`kubectl rolling-update` 在执行的过程会显示进度：

```
Scaling up my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 from 0 to 3, scaling down my-nginx from 3 to 0 (keep 3 pods available, don't exceed 4 pods)
Scaling my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 up to 1
Scaling my-nginx down to 2
Scaling my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 up to 2
Scaling my-nginx down to 1
Scaling my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 up to 3
Scaling my-nginx down to 0
Update succeeded. Deleting old controller: my-nginx
Renaming my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 to my-nginx
replicationcontroller "my-nginx" rolling updated
```


如果您遇到了问题，您可以在中途停止滚动升级并且可以使用 `--rollback` 回滚到上个版本。

```shell
$ kubectl rolling-update my-nginx --rollback
Setting "my-nginx" replicas to 1
Continuing update with existing controller my-nginx.
Scaling up nginx from 1 to 1, scaling down my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 from 1 to 0 (keep 1 pods available, don't exceed 2 pods)
Scaling my-nginx-ccba8fbd8cc8160970f63f9a2696fc46 down to 0
Update succeeded. Deleting my-nginx-ccba8fbd8cc8160970f63f9a2696fc46
replicationcontroller "my-nginx" rolling updated
```


这是容器不变性是巨大财富的一个例子。

如果您需要更新的不仅仅是镜像（例如命令参数，环境变量），您可以用一个新的名称和不同的 label 来创建一个新的 replication controller ，例如：


```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx-v4
spec:
  replicas: 5
  selector:
    app: nginx
    deployment: v4
  template:
    metadata:
      labels:
        app: nginx
        deployment: v4
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.2
        args: ["nginx", "-T"]
        ports:
        - containerPort: 80
```


然后进行发布：

```shell
$ kubectl rolling-update my-nginx -f ./nginx-rc.yaml
Created my-nginx-v4
Scaling up my-nginx-v4 from 0 to 5, scaling down my-nginx from 4 to 0 (keep 4 pods available, don't exceed 5 pods)
Scaling my-nginx-v4 up to 1
Scaling my-nginx down to 3
Scaling my-nginx-v4 up to 2
Scaling my-nginx down to 2
Scaling my-nginx-v4 up to 3
Scaling my-nginx down to 1
Scaling my-nginx-v4 up to 4
Scaling my-nginx down to 0
Scaling my-nginx-v4 up to 5
Update succeeded. Deleting old controller: my-nginx
replicationcontroller "my-nginx-v4" rolling updated
```



## 故障排除

如果在滚动升级过程中达到了 `timeout` 时间，这个操作会失败，造成一些 pod 属于新的 replication controller ，一些属于原来的 replication controller。

要从失败的地方继续进行，可以使用相同的命令重试。

如果在尝试更新前回滚到原来的状态，在原来的命令后面加上 `--rollback=true`。这样就会回滚所有的变更。