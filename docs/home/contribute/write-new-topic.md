---
cn-approvers:
- tianshapjq
title: 编写一个新的主题
---


{% capture overview %}

本页面展示如何为 Kubernetes 文档创建一个新的主题。
{% endcapture %}

{% capture prerequisites %}

按照 [创建一个文档 Pull Request](/docs/home/contribute/create-pull-request/) 中的方法来创建一个 Kubernetes 文档库的分支。
{% endcapture %}

{% capture steps %}


## 选择一个页面类型


在您开始编写新主题之前，先思考以下哪个页面类型最适合您的内容：

<table>

  <tr>

    <td>任务</td>    <td>任务页面展示如何做一件事情。中心思想是当读者在阅读页面时，能够知道他们能够做的一系列步骤。任务页面长度没有限制，只需要专注于一个领域。在任务页面中，可以将概要说明和详细步骤混合在一起，但是如果您是提供一个冗长的说明，您需要在概念主题中做这个事情。相关的任务和概念需要相互链接。短任务页面示例，参见 <a href="/docs/tasks/configure-pod-container/configure-volume-storage/">配置 Pod 使用卷作为存储</a>。长任务页面示例，参见 <a href="/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/">配置 Liveness 和 Readiness 探针</a></td>
  </tr>

  <tr>

    <td>教程</td>
    <td>教程页面展示了如何完成将多个 Kubernetes 功能连接在一起的目标。教程可能会提供读者在阅读页面时可以实际执行的几个步骤序列， 或者可能提供相关代码段的解释。例如，教程可以提供代码示例的演练，教程可以把 Kubernetes 的相关特性结合起来做一个简要说明，但每个特性应链接到相关的概念主题，以便对特性做更深入的解释。</td>
  </tr>

  <tr>

    <td>概念</td>
    <td>概念页面解释了 Kubernetes 的一些概念。例如，概念页面可能会描述 Kubernetes Deployment 对象，并解释它在部署，伸缩和更新应用程序时扮演的角色。 通常，概念页面不包括步骤序列，而是提供指向任务或教程的链接。概念的示例，参见 <a href="/docs/concepts/architecture/nodes/">Nodes</a>。</td>
  </tr>

</table>


当您编写主题时，每个页面类型都有一个 [模板](/docs/home/contribute/page-templates/) 可供使用。使用模板有助于确保给定类型主题之间的一致性。


## 选择一个标题和文件名


选择一个标题，其中包含您希望搜索引擎找到的关键字。
创建一个文件名，使用由连字符分隔的标题中的单词。例如，标题为 [使用 HTTP 代理来访问 Kubernetes API （Using an HTTP Proxy to Access the Kubernetes API）](/docs/tasks/access-kubernetes-api/http-proxy-access-api/) 可以使用 `http-proxy-access-api.md` 作为文件名。您不需要在文件名中增加 "kubernetes"，因为 "kubernetes" 已经存在主题的 URL 中，例如：

       http://kubernetes.io/docs/tasks/access-kubernetes-api/http-proxy-access-api/


## 将主题的标题添加到 front matter


在您的主题中，将 `title` 字段添加到 [front matter](https://jekyllrb.com/docs/frontmatter/)。front matter 是在页面顶部的三条虚线形成的 YAML 块。以下是一个示例：

    ---
    title: Using an HTTP Proxy to Access the Kubernetes API
    ---


## 选择一个目录


根据您的页面类型，将您创建的文件放到以下其中一个目录的子目录中：

* /docs/tasks/
* /docs/tutorials/
* /docs/concepts/


您可以将文件放到一个已存在的子目录，也可以创建一个新的子目录。


## 在内容表格中添加一个入口


根据页面类型，在以下其中一个文件中创建一个入口：

* /_data/tasks.yaml
* /_data/tutorials.yaml
* /_data/concepts.yaml


以下是在 /_data/tasks.yaml 中的一个入口示例：

    - docs/tasks/configure-pod-container/configure-volume-storage.md


## 引用其它文件的代码


如果想要在您的主题中引用一个代码文件，首先将代码文件放到 Kubernetes 文档库中，最好和您的主题文件同一目录。然后在您的主题文件中，使用 `include` 标签：

<pre>&#123;% include code.html language="&lt;LEXERVALUE&gt;" file="&lt;RELATIVEPATH&gt;" ghlink="/&lt;PATHFROMROOT&gt;" %&#125;</pre>


其中：


* `<LEXERVALUE>` 指代码文件使用的语言。必须是
[Rouge 支持的类型](https://github.com/jneen/rouge/wiki/list-of-supported-languages-and-lexers).
* `<RELATIVEPATH>` 指引入文件的路径，使用与当前文件的相对路径，例如 `local-volume.yaml`。
* `<PATHFROMROOT>` 指文件相对于 root 的路径，例如，`docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/local-volumes.yaml`.


以下是使用 `include` 标签的示例：

<pre>&#123;% include code.html language="yaml" file="gce-volume.yaml" ghlink="/docs/tutorials/stateful-application/gce-volume.yaml" %&#125;</pre>


## 展示如何通过一个配置文件创建一个 API 对象


如果您想要向读者展示如何基于一个配置文件来创建一个 API 对象，需要把配置文件放到 Kubernetes 文档库中，最好和您的主题文件同一目录。


然后在您的主题中，使用如下命令：

    kubectl create -f https://k8s.io/<PATHFROMROOT>


其中 `<PATHFROMROOT>` 是对于 root 的相对路径，例如，`docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/local-volumes.yaml`。


以下是一个通过配置文件创建一个 API 对象的命令示例：

    kubectl create -f https://k8s.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/local-volumes.yaml


作为一个使用该技术的例子，请见 [Running a Single-Instance Stateful Application](/docs/tutorials/stateful-application/run-stateful-application/)


## 在主题中添加图片


把图片文件放到 `/images` 目录，文件最好是 SVG 格式。

{% endcapture %}

{% capture whatsnext %}

* 学习 [使用页面模板](/docs/home/contribute/page-templates/)。
* 学习 [展示您对文档的修改](/docs/home/contribute/stage-documentation-changes/)。
* 学习 [创建一个 pull request](/docs/home/contribute/create-pull-request/)。
{% endcapture %}

{% include templates/task.md %}
