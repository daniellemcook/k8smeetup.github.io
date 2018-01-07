---
approvers:
- chenopis
cn-approvers:
- zhangqx2010
title: 选项卡示例
---



您可以在这个网站 Markdown 页面（.md file）中添加选项卡组，用于显示解决方案的多种方法。


## 示例


{% capture default_tab %}
选择一个选项卡。
{% endcapture %}


{% capture calico %}
```shell
kubectl apply -f "http://docs.projectcalico.org/v2.4/getting-started/kubernetes/installation/hosted/kubeadm/calico.yaml"
```
{% endcapture %}

{% capture flannel %}
```shell
kubectl apply -f "https://github.com/coreos/flannel/blob/master/Documentation/kube-flannel.yml?raw=true"
```
{% endcapture %}

{% capture romana %}
```shell
kubectl apply -f "https://raw.githubusercontent.com/romana/romana/master/containerize/specs/romana-kubeadm.yml"
```
{% endcapture %}

{% capture weave_net %}
```shell
kubectl apply -f "https://git.io/weave-kube"
```
{% endcapture %}

{% assign tab_names = "Default,Calico,Flannel,Romana,Weave Net" | split: ',' | compact %}
{% assign tab_contents = site.emptyArray | push: default_tab | push: calico | push: flannel | push: romana | push: weave_net %}

{% include tabs.md %}


## 选项卡的 Liquid 模板代码示例


以下是上述选项卡示例的 [Liquid](https://shopify.github.io/liquid/) 模板代码，以说明如何指定每个选项卡的内容。 将 [`/_includes/tabs.md`](https://git.k8s.io/kubernetes.github.io/_includes/tabs.md) 包含在文件最后，然后使用这些元素来呈现实际的选项卡集。


### 代码

````liquid
{{ "{% capture default_tab " }}%}
Select one of the tabs.
{{ "{% endcapture " }}%}

{{ "{% capture calico " }}%}
```shell
kubectl apply -f "http://docs.projectcalico.org/v2.4/getting-started/kubernetes/installation/hosted/kubeadm/calico.yaml"
```
{{ "{% endcapture " }}%}

{{ "{% capture flannel " }}%}
```shell
kubectl apply -f "https://github.com/coreos/flannel/blob/master/Documentation/kube-flannel.yml?raw=true"
```
{{ "{% endcapture " }}%}

{{ "{% capture romana " }}%}
```shell
kubectl apply -f "https://raw.githubusercontent.com/romana/romana/master/containerize/specs/romana-kubeadm.yml"
```
{{ "{% endcapture " }}%}

{{ "{% capture weave_net " }}%}
```shell
kubectl apply -f "https://git.io/weave-kube"
```
{{ "{% endcapture " }}%}

{{ "{% assign tab_names = 'Default,Calico,Flannel,Romana,Weave Net' | split: ',' | compact " }}%}
{{ "{% assign tab_contents = site.emptyArray | push: default_tab | push: calico | push: flannel | push: romana | push: weave_net " }}%}

{{ "{% include tabs.md " }}%}
````


### 捕捉标签页内容

````liquid
{{ "{% capture calico " }}%}
```shell
kubectl apply -f "http://docs.projectcalico.org/v2.4/getting-started/kubernetes/installation/hosted/kubeadm/calico.yaml"
```
{{ "{% endcapture " }}%}
````


`capture [variable_name]` 标记存储文本或 markdown 内容，并将它们分配给指定的变量。


### 分配选项卡名

````liquid
{{ "{% assign tab_names = 'Default,Calico,Flannel,Romana,Weave Net' | split: ',' | compact " }}%}
````


`assign tab_names` 标签带有一个用于标签的标签列表。标签文字可以包含空格。给定的逗号分隔的字符串被分割成一个数组并分配给 `tab_names` 变量。


### 分配选项卡内容

````liquid
{{ "{% assign tab_contents = site.emptyArray | push: default_tab | push: calico | push: flannel | push: romana | push: weave_net " }}%}
````


`assign tab_contents` 标签将上面捕获的每个选项卡窗格的内容作为元素添加到 `tab_contents` 数组中。


### 包含 tabs.md 模版

````liquid
{{ "{% include tabs.md " }}%}
````


`{{ "{％include tabs.md " }}％}` 拉入选项卡模板代码，它使用 `tab_names` 和 `tab_contents` 变量来呈现选项卡集。
