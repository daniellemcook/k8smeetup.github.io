---
cn-approvers:
- lichuqiang
title: Kubernetes 对象管理
---


{% capture overview %}

`kubectl` 命令行工具支持多种不同的创建和管理 Kubernetes 对象的方法。
本文提供了对不同方法的概述。
{% endcapture %}

{% capture body %}


## 管理技术

**警告：** 一种 Kubernetes 对象只应该用一种技术来管理。
针对同一对象的技术混合和竞争会导致未定义的行为。

| 管理技术        | 作用于     |推荐环境    | 支持的编写工具     | 学习曲线   |
|----------------|-----------|-----------|------------------|------------|
| 命令式的指令     | 活动对象   | 开发项目   | 1+               | 最低       |
| 命令式对象配置   | 单个文件   | 生产项目   | 1                | 中等       |
| 声明式对象配置   | 文件目录   | 生产项目   | 1+               | 最高       |


##  命令式的指令

当使用命令式的指令时，用户直接在集群中的活动对象上操作。
用户提供操作，作为 `kubectl` 命令的参数或标记。


这是在集群中启动或运行一次性任务最简单的方法。
由于这种技术直接作用于活动对象，所以它不提供先前配置的历史。


### 示例

通过创建一个 Deployment 对象来运行一个 nginx 容器的实例：

```sh
kubectl run nginx --image nginx
```


使用不同的语法做同样的事：

```sh
kubectl create deployment nginx --image nginx
```


### 权衡

与对象配置相比的优点：


- 命令简单，易于学习，易于记忆。
- 命令只需要一个步骤就可以完成对集群的修改。


与对象配置相比的缺点：

- 命令没有与修改审查流程进行集成。
- 命令不提供与修改相关的审计跟踪。
- 除了活动对象，命令不提供记录来源。
- 命令不提供创建新对象的模板。


## 命令式对象配置

在命令式对象配置中，kubectl 命令指定操作（创建、替换等）、可选参数和至少一个文件名称。
指定文件必须以 YAML 或 JSON 格式包含对象的完整定义。


查看 [API 参考](/docs/api-reference/{{page.version}}/)
了解更多对象定义相关的详情。


**警告：** 命令式的 `replace` 命令会用新提供的 sepc 定义替换当前的 spec，
同时移除配置文件中缺失的所有对对象的修改。 该方法不应该被用于那些 spec 独立于配置文件进行更新的资源类型。
例如，`LoadBalancer` 类型的 Service 资源，它们的 `externalIPs` 字段是独立于配置，由集群进行更新的。


### 示例

创建在配置文件中定义的对象：

```sh
kubectl create -f nginx.yaml
```


删除两个配置文件中定义的对象：

```sh
kubectl delete -f nginx.yaml -f redis.yaml
```


通过重写实时配置，更新配置文件中定义的对象。

```sh
kubectl replace -f nginx.yaml
```


### 权衡

与命令式指令相比的优点：

- 对象配置可以存储在类似 Git 的源代码控制系统中。
- 对象配置可以和类似推送前的审查，以及审计跟踪等流程进行集成。
- 对象配置提供了创建新对象的模板。


与命令式指令相比的缺点：

- 对象配置需要对对象模式有基本的了解。
- 对象配置需要编写 YAML 文件的额外步骤。


与声明式对象配置相比的优点：

- 命令式对象配置行为更简单，更易于理解。
- 在 Kubernetes 1.5 版本中，命令式对象配置更加成熟。


与声明式对象配置相比的缺点：

- 命令式对象配置最适合于文件，而不是目录。
- 活动对象的更新必须反映在配置文件中，否则它们将在下次替换时丢失。


## 声明式对象配置

使用声明式对象配置时，用户操作本地存储的对象配置文件，然而用户不会定义对文件的操作。
创建、更新和删除操作由 `kubectl` 逐个对象自动进行检测。 这使得该技术能够作用于目录，
目录中的不同对象可能需要不同的操作。


**注意：** 声明式对象配置保留了其他修改者所做的修改，即使这些修改没有合并到对象配置文件中。
通过使用 `patch` API 操作来只写入观察到的差异，而不是使用 `replace` API
操作来替换整个对象配置，可以做到这一点。


### 示例

处理 `configs` 目录中的所有对象配置文件，并创建或对活动对象打补丁（patch）:

```sh
kubectl apply -f configs/
```


递归地处理目录：

```sh
kubectl apply -R -f configs/
```


### 权衡

与命令式对象配置相比的优点：

- 直接对活动对象进行的修改被保留，即使这些修改没有合并到对象配置文件中。
- 声明式对象配置可以更好地支持对目录的操作，并自动逐个对象检测操作类型（创建、 打补丁或删除）。


与命令式对象配置相比的缺点：

- 在意想不到的情况下，声明式对象配置很难调试，其结果也很难理解。
- 使用区别对比进行部分更新，造成了复杂的合并和打补丁操作。

{% endcapture %}

{% capture whatsnext %}

- [使用命令式指令管理 Kubernetes 对象](/docs/concepts/overview/object-management-kubectl/imperative-command/)
- [使用对象配置（命令式）管理 Kubernetes 对象](/docs/concepts/overview/object-management-kubectl/imperative-config/)
- [使用对象配置（声明式）管理 Kubernetes 对象](/docs/concepts/overview/object-management-kubectl/declarative-config/)
- [Kubectl 命令参考](/docs/user-guide/kubectl/{{page.version}}/)
- [Kubernetes API 参考](/docs/api-reference/{{page.version}}/)

{% comment %}
{% endcomment %}
{% endcapture %}

{% include templates/concept.md %}
