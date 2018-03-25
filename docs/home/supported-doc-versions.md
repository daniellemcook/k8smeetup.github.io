---
title: 支持的 Kubernetes 文档的版本
cn-approvers:
- chentao1596
---


{% capture overview %}


该网站包含了当前版本及以前四个版本的 Kubernetes 的文档。

{% endcapture %}

{% capture body %}


## 当前版本

当前版本是
[{{page.version}}](/)。


## 早期版本

{% for v in page.versions %}
{% if v.version != page.version %}
* [{{ v.version }}]({{v.url}})
{% endif %}
{% endfor %}

{% endcapture %}

{% include templates/concept.md %}
