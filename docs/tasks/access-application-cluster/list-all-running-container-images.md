---
cn-approvers:
- zhangqx2010
title: 在运行集群中查看所有 Container Image
---


{% capture overview %}


本文说明如何使用 kubectl 查看集群中所有运行的 Pod 所使用的 Container Image。

{% endcapture %}

{% capture prerequisites %}

{% include task-tutorial-prereqs.md %}

{% endcapture %}

{% capture steps %}


这个练习中将使用 kubectl 获取运行在集群中所有的 Pod，并且格式化地输出每个 Pod 包含的 Container。


## 查看所有 namespace 中所有的 Container


- 使用 `kubectl get pods --all-namespaces` 查看所有 namespace 中的 Pod
- 使用 `-o jsonpath={..image}` 参数将输出格式调整为只包含 Container image 的名称。
  这将递归地取出返回 JSON 中地 `image` 字段。
  - 关于如何使用 jsonpath 的更多信息，参见 [jsonpath reference](/docs/user-guide/jsonpath/)
- 使用标准工具调整输出格式：`tr`，`sort`，`uniq`
  - 使用 `tr` 将空格替换为换行符
  - 使用 `sort` 对结果排序
  - 使用 `uniq` 对镜像的数量计数

```sh
kubectl get pods --all-namespaces -o jsonpath="{..image}" |\
tr -s '[[:space:]]' '\n' |\
sort |\
uniq -c
```


上面的命令会对所有对返回项递归地取出 `image` 字段并计数。


或者，使用 Pod 中 image 字段的绝对路径。这可以确保得到正确的字段，即使字段名是重复的，
例如，在一个 item 中有很多字段名都是 `name`:

```sh
kubectl get pods --all-namespaces -o jsonpath="{.items[*].spec.containers[*].image}"
```


jsonpath 解析如下：

- `.items[*]`：每一个返回的值
- `.spec`：取 spec
- `.containers[*]`：每一个 container
- `.image`：取 image


**注意：** 当使用名称获取单个 Pod 时，例如，`kubectl get pod nginx`，
应该省略路径中的 `.items[*]` 部分，因为返回的是单个 Pod 而不是 item 列表。


## 按 Pod 查看 Container

通过使用 `range` 遍历单独的 element 以更好的格式化输出。

```sh
kubectl get pods --all-namespaces -o=jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' |\
sort
```


## 使用 Pod label 过滤查看 Container

使用 -l 标记查看只匹配特定 label 的 Pod。下面的查询只匹配具有 `app=nginx` label 的 Pod。

```sh
kubectl get pods --all-namespaces -o=jsonpath="{..image}" -l app=nginx
```


## 使用 Pod 所在的 namespace 过滤查看 Container

使用 namespace 标记查看只属于特定 namespace 的 Pod。下面的查询只会匹配 `kube-system` namespace 中的 Pod。

```sh
kubectl get pods --namespace kube-system -o jsonpath="{..image}"
```


## 使用 go-template 而非 jsonpath 查看 Container

除了 jsonpath 外，Kubectl 支持使用 [go-templates](https://golang.org/pkg/text/template/) 格式化输出：

{% raw %}
```sh
kubectl get pods --all-namespaces -o go-template --template="{{range .items}}{{range .spec.containers}}{{.image}} {{end}}{{end}}"
```
{% endraw %}


{% endcapture %}

{% capture discussion %}

{% endcapture %}

{% capture whatsnext %}


### 参考

* [Jsonpath](/docs/user-guide/jsonpath/) 参考手册
* [Go template](https://golang.org/pkg/text/template/) 参考手册

{% endcapture %}

{% include templates/task.md %}
