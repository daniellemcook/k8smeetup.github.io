---
title: 使用 kubectl patch 更新 API 对象
description: 使用 kubectl patch 更新 Kubernetes 对象。做一个策略性合并补丁或一个 JSON 合并补丁。
---


{% capture overview %}


本文说明了如何使用 `kubectl patch` 更新 API 对象。本文中的例子演示了一个策略性合并补丁和一个 JSON 合并补丁。

{% endcapture %}

{% capture prerequisites %}

{% include task-tutorial-prereqs.md %}

{% endcapture %}


{% capture steps %}


## 使用策略性合并补丁更新一个 Deployment

这里有一个副本数为 2 的 Deployment 配置文件，每个副本是一个拥有单个容器的 Pod ：

{% include code.html language="yaml" file="deployment-patch-demo.yaml" ghlink="/docs/tasks/run-application/deployment-patch-demo.yaml" %}


创建这个 Deployment ：

```shell
kubectl create -f https://k8s.io/docs/tasks/run-application/deployment-patch-demo.yaml
```


查看和这个 Deployment 关联的 Pod ：

```shell
kubectl get pods
```


输出内容显示这个 Deployment 拥有两个 Pod 。`1/1` 表示每个 Pod 拥有一个容器：


```
NAME                        READY     STATUS    RESTARTS   AGE
patch-demo-28633765-670qr   1/1       Running   0          23s
patch-demo-28633765-j5qs3   1/1       Running   0          23s
```


记下这些运行中的 Pod 的名称，稍后您会看到这些 Pod 被中止了并且会被新的 Pod 所取代。


此时，每个 Pod 拥有一个运行 nginx 镜像的容器。现在假设您希望每个 Pod 拥有两个容器：一个运行 nginx，另一个运行 redis。


创建一个文件 `patch-file.yaml` ，内容如下：

```shell
spec:
  template:
    spec:
      containers:
      - name: patch-demo-ctr-2
        image: redis
```


给这个 Deployment 打补丁：

```shell
kubectl patch deployment patch-demo --patch "$(cat patch-file.yaml)"
```


查看打补丁后的 Deployment：

```shell
kubectl get deployment patch-demo --output yaml
```


输出内容显示这个 Deployment 的 PodSpec 中有两个容器：

```shell
containers:
- image: redis
  imagePullPolicy: Always
  name: patch-demo-ctr-2
  ...
- image: nginx
  imagePullPolicy: Always
  name: patch-demo-ctr
  ...
```


查看打补丁后的 Deployment 所关联的 Pod ：

```shell
kubectl get pods
```


输出内容显示当前正在运行的 Pod 名称与打补丁之前的 Pod 名称不一样了。这个 Deployment 中止了原先的 Pod 然后使用更新后的 spec 重新创建了两个新的 Pod 。`2/2` 表示每个 Pod 拥有两个容器：

```
NAME                          READY     STATUS    RESTARTS   AGE
patch-demo-1081991389-2wrn5   2/2       Running   0          1m
patch-demo-1081991389-jmg7b   2/2       Running   0          1m
```


让我们仔细看看这个补丁例子中的其中一个 Pod ：

```shell
kubectl get pod <your-pod-name> --output yaml
```


输出内容显示这个 Pod 拥有两个容器，一个运行 nginx ，一个运行 redis ：

```
containers:
- image: redis
  ...
- image: nginx
  ...
```


### 关于策略性合并补丁的说明


通过补丁，您可以避免定义整个对象，只需要定义您希望更改的部分。比如：在上述例子中，您只需要在 `PodSpec` 中 `containers` 列表中定义了一个容器。


上述例子中的补丁称为 *策略性合并补丁* 。通过策略性合并补丁，您只需要定义新增的元素就可以更新一个列表。列表中已有的元素仍然保留，新增的元素和已有的元素会被合并。上述例子中，最终结果的 `containers` 列表中既有原先的 nginx 容器，也有新增的 redis 容器。


## 使用一个 JSON 合并补丁更新一个 Deployment


策略性合并补丁与 [JSON 合并补丁](https://tools.ietf.org/html/rfc6902) 是不一样的。如果您希望使用 JSON 合并补丁更新一个列表，您必须重新定义整个列表。新的列表会完全替换掉原先的列表。


`kubectl patch` 命令拥有一个 `type` 参数，您可以将其设置为以下值：

<table>
  <tr><th>参数值</th><th>合并类型</th></tr>
  <tr><td>json</td><td><a href="https://tools.ietf.org/html/rfc6902">JSON 补丁, RFC 6902</a></td></tr>
  <tr><td>merge</td><td><a href="https://tools.ietf.org/html/rfc7386">JSON 合并补丁, RFC 7386</a></td></tr>
  <tr><td>strategic</td><td>策略性合并补丁</td></tr>
</table>


关于 JSON 补丁 与 JSON 合并补丁的区别，请查看 [JSON 补丁 与 JSON 合并补丁](http://erosb.github.io/post/json-patch-vs-merge-patch/)。


`type` 参数的默认值是 `strategic`。因此，在上述例子中，您打了一个策略性合并补丁。


接下来，继续使用上述的 Deployment 做一个 JSON 合并补丁。创建一个名为 `patch-file-2.yaml` 的文件，内容如下：

```shell
spec:
  template:
    spec:
      containers:
      - name: patch-demo-ctr-3
        image: gcr.io/google-samples/node-hello:1.0
```


在 patch 命令中，设置参数 `type` 的值为 `merge` ：

```shell
kubectl patch deployment patch-demo --type merge --patch "$(cat patch-file-2.yaml)"
```


查看打补丁后的 Deployment ：

```shell
kubectl get deployment patch-demo --output yaml
```


在这个补丁中您定义的 `containers` 列表只有一个容器。输出内容显示这个只有一个容器的 `containers` 列表把原先的 `containers` 列表替换了。

```shell
spec:
  containers:
  - image: gcr.io/google-samples/node-hello:1.0
    ...
    name: patch-demo-ctr-3
```


列出运行中的 Pod ：

```shell
kubectl get pods
```


在输出内容中，您可以看到原先的 Pod 被中止了，然后有新的 Pod 被创建了。`1/1` 表示每个 Pod 只拥有一个容器。

```shell
NAME                          READY     STATUS    RESTARTS   AGE
patch-demo-1307768864-69308   1/1       Running   0          1m
patch-demo-1307768864-c86dc   1/1       Running   0          1m
```


## kubectl patch 命令的其它形式


`kubectl patch` 命令接受 YAML 或 JSON 格式的补丁，且补丁能够以文件或直接以命令行参数的形式进行传递。


创建一个名为 `patch-file.json` 的文件，内容如下：

```shell
{
   "spec": {
      "template": {
         "spec": {
            "containers": [
               {
                  "name": "patch-demo-ctr-2",
                  "image": "redis"
               }
            ]
         }
      }
   }
}
```


以下的命令是等价的：


```shell
kubectl patch deployment patch-demo --patch "$(cat patch-file.yaml)"
kubectl patch deployment patch-demo --patch $'spec:\n template:\n  spec:\n   containers:\n   - name: patch-demo-ctr-2\n     image: redis'

kubectl patch deployment patch-demo --patch "$(cat patch-file.json)"
kubectl patch deployment patch-demo --patch '{"spec": {"template": {"spec": {"containers": [{"name": "patch-demo-ctr-2","image": "redis"}]}}}}'
```


## 总结


在这个例子中，您使用 `kubectl patch` 命令对一个 Deployment 的实时配置进行了更新。您并没有改变原先用来创建 Deployment 对象的那个配置文件。其它用于更新 API 对象的命令有：
[kubectl annotate](/docs/user-guide/kubectl/{{page.version}}/#annotate),
[kubectl edit](/docs/user-guide/kubectl/{{page.version}}/#edit),
[kubectl replace](/docs/user-guide/kubectl/{{page.version}}/#replace),
[kubectl scale](/docs/user-guide/kubectl/{{page.version}}/#scale),
and
[kubectl apply](/docs/user-guide/kubectl/{{page.version}}/#apply).

{% endcapture %}


{% capture whatsnext %}


* [Kubernetes 对象管理](/docs/tutorials/object-management-kubectl/object-management/)
* [使用命令式命令管理 Kubernetes 对象](/docs/tutorials/object-management-kubectl/imperative-object-management-command/)
* [使用配置文件对 Kubernetes 对象进行命令式管理](/docs/tutorials/object-management-kubectl/imperative-object-management-configuration/)
* [使用配置文件对 Kubernetes 对象进行声明式管理](/docs/tutorials/object-management-kubectl/declarative-object-management-configuration/)

{% endcapture %}

{% include templates/task.md %}
