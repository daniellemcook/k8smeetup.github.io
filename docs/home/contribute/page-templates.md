---
title: 使用页面模板
cn-approvers:
- chentao1596
---





<p>对于想要在下面这些 Kubernetes 文档中提交新主题的贡献者，可以使用下面几种页面模板：</p>


<ul>
    <li><a href="#task_template">任务</a></li>
    <li><a href="#tutorial_template">指南</a></li>
    <li><a href="#concept_template">概念</a></li>
</ul>


<p>页面模板在 <a href="https://github.com/kubernetes/kubernetes.github.io">kubernetes.github.io</a> 仓库的 <a href="https://git.k8s.io/kubernetes.github.io/_includes/templates" target="_blank">_includes/templates</a> 目录下。


<h2 id="task_template">任务模板</h2>


<p>任务页面通常是通过给出一个简短的步骤序列展示如何做一件事情。任务页面进行解释的内容很少，但通常提供指向概念主题的链接，这些概念主题提供了相关的背景和知识。</p>


<p>想要写一个新的任务页面，请在 /docs/tasks 的子目录下创建一个 Markdown 文件。在您的 Markdown 文件中，为如下变量提供相应的值：</p>


<ul>
    <li>overview - 必选</li>
    <li>prerequisites - 必选</li>
    <li>steps - 必选</li>
    <li>discussion - 可选</li>
    <li>whatsnext - 可选</li>
</ul>


<p>然后像下面这样引入 templates/task.md：</p>

{% raw %}<pre>...
{% include templates/task.md %}</pre>{% endraw %}


<p>在 <code>steps</code> 部分，使用 <code>##</code> 开始一个二级标题。对于子标题，根据需要使用 <code>###</code> 和 <code>####</code>。类似地，如果您的页面想有一个 <code>讨论</code> 章节，也请使用一个二级标题开始。</p>


<p>下面是一个使用任务模板的 Markdonw 文件示例：</p>


{% raw %}
<pre>---
title: 配置这件事情
---

{% capture overview %}
本页展示怎么样 ...
{% endcapture %}

{% capture prerequisites %}
* 执行此操作。
* 再执行此操作。
{% endcapture %}

{% capture steps %}
## 做 ...

1. 执行此操作。
1. 接下来执行此操作。可以阅读这些 [相关说明](...)。
{% endcapture %}

{% capture discussion %}
## 理解 ...

理解您刚才执行的步骤是一件有趣的事情。
{% endcapture %}

{% capture whatsnext %}
* 了解更多 [这](...)。
* 查看 [相关的任务](...)。
{% endcapture %}

{% include templates/task.md %}</pre>
{% endraw %}


<p>下面是一个已发布的主题的示例，该示例使用了任务模板：</p>


<p><a href="/docs/tasks/access-kubernetes-api/http-proxy-access-api">使用 HTTP 代理访问 Kubernetes API</a></p>


<h2 id="tutorial_template">指南模板</h2>


<p>指南页面展示了如何完成一个包含多个任务的目标。一般来说，指南页面有多个部分，每个部分都包含一系列步骤。例如，指南可以提供一个代码示例的演练，该示例展示了 Kubernetes 的某些功能。指南可以只包含浅显的解释，但应链接到相关概念主题以进行深入的解释。


<p>想要写一个新的指南页面，请在 /docs/tutorials 的子目录下创建一个 Markdown 文件。在您的 Markdown 文件中，为如下变量提供相应的值：</p>


<ul>
    <li>overview - 必选</li>
    <li>prerequisites - 必选</li>
    <li>objectives - 必选</li>
    <li>lessoncontent - 必选</li>
    <li>cleanup - 可选</li>
    <li>whatsnext - 可选</li>
</ul>


<p>然后像下面这样引入 templates/tutorial.md：</p>

{% raw %}<pre>...
{% include templates/tutorial.md %}</pre>{% endraw %}


<p>在 <code>LessonContent</code> 部分，使用 <code>##</code> 开始一个二级标题。对于子标题，根据需要使用 <code>###</code> 和 <code>####</code>。


<p>下面是一个使用指南模板的 Markdonw 文件示例：</p>


{% raw %}
<pre>---
title: 运行这件事情
---

{% capture overview %}
本页展示怎么样 ...
{% endcapture %}

{% capture prerequisites %}
* 执行此步骤。
* 再执行该步骤。
{% endcapture %}

{% capture objectives %}
* 学习它。
* 构建它。
* 运行它。
{% endcapture %}

{% capture lessoncontent %}
## 构建 ...

1. 执行此操作。
1. 接下来执行该操作。可以阅读这些 [相关说明](...)。

## 运行 ...

1. 执行此步骤。
1. 接下来执行该步骤。

## 理解代码
理解前面步骤中运行的代码是一件有趣的事情
{% endcapture %}

{% capture cleanup %}
* 删除它。
* 停止它。
{% endcapture %}

{% capture whatsnext %}
* 了解更多 [这](...)。
* 查看 [相关的指南](...)。
{% endcapture %}

{% include templates/tutorial.md %}</pre>
{% endraw %}


<p>下面是一个已发布的主题的示例，该示例使用了指南模板：</p>


<p><a href="/docs/tutorials/stateless-application/run-stateless-application-deployment/">使用 Deployment 运行一个无状态的应用</a></p>


<h2 id="concept_template">概念模板</h2>


<p>概念页面解释 Kubernetes 的某些方面。例如，概念页面可以描述 Kubernetes 的 Deployment 对象，并解释它在部署、缩放和更新应用程序时所扮演的角色。通常，概念页面不包括步骤序列，而是提供到任务或指南的链接。


<p>想要写一个新的概念页面，请在 /docs/concepts 的子目录下创建一个 Markdown 文件。在您的 Markdown 文件中，为如下变量提供相应的值：</p>


<ul>
    <li>overview - 必选</li>
    <li>body - 必选</li>
    <li>whatsnext - 可选</li>
</ul>


<p>然后像下面这样引入 templates/concept.md</p>

{% raw %}<pre>...
{% include templates/concept.md %}</pre>{% endraw %}


<p>在 <code>body</code> 部分，使用 <code>##</code> 开始一个二级标题。对于子标题，根据需要使用 <code>###</code> 和 <code>####</code>。


<p>下面是一个使用概念模板的 Markdonw 文件示例：</p>


{% raw %}
<pre>---
title: 理解这件事情
---

{% capture overview %}
本页说明 ...
{% endcapture %}

{% capture body %}
## 理解 ...

Kubernetes 提供 ...

## 使用 ...

为了使用 ...
{% endcapture %}

{% capture whatsnext %}
* 了解更多 [这](...)。
* 查看 [相关的任务](...)。
{% endcapture %}

{% include templates/concept.md %}</pre>
{% endraw %}


<p>下面是一个已发布的主题的示例，该示例使用了概念模板：</p>

<p><a href="/docs/concepts/overview/working-with-objects/annotations/">Annotations</a></p>



