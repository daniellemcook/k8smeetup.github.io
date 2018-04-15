---
cn-approvers:
- jonyhy96
title: 为Kubernetes Federation API生成参考文档
---



{% capture overview %}



本页面将展示如何自动为Kubernetes Federation API生成相关参考文档。

{% endcapture %}


{% capture prerequisites %}



* 你需要安装好[Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

* 你需要安装1.9.1 或者更高版本的[Golang](https://golang.org/doc/install),同时你的环境变量中需要将`$GOPATH`设置正确

* 你需要知道如何去创建一个`PR`.例如,你首先需要将远程仓库`fork`到你的个人仓库.详见[创建一个文档相关的PR](/docs/home/contribute/create-pull-request/)

{% endcapture %}


{% capture steps %}




## 运行update-federation-api-docs.sh脚本

如果你本地还没有Kubernetes federation源码,请先下载：

```shell
mkdir $GOPATH/src
cd $GOPATH/src
go get github.com/kubernetes/federation
```

确定你[kubernetes/federation](https://github.com/kubernetes/federation)项目的根目录.例如,如果你执行了上面的命令，你的根目录应该为`$GOPATH/src/github.com/kubernetes/federation`下文将以`<fed-base>`指代项目根目录.

运行文档自动生成脚本:

```shell
cd <fed-base>
hack/update-federation-api-reference-docs.sh
```

脚本将运行
[gcr.io/google_containers/gen-swagger-docs](https://console.cloud.google.com/gcr/images/google-containers/GLOBAL/gen-swagger-docs?gcrImageListquery=%255B%255D&gcrImageListpage=%257B%2522t%2522%253A%2522%2522%252C%2522i%2522%253A0%257D&gcrImageListsize=50&gcrImageListsort=%255B%257B%2522p%2522%253A%2522uploaded%2522%252C%2522s%2522%253Afalse%257D%255D)
镜像生成以下相关文档:

* /docs/api-reference/extensions/v1beta1/operations.html
* /docs/api-reference/extensions/v1beta1/definitions.html
* /docs/api-reference/v1/operations.html
* /docs/api-reference/v1/definitions.html

自动生成的文档不会自动发布,你需要手动将他们拷贝到
[kubernetes/website](https://github.com/kubernetes/website/tree/master/docs/reference/generated)
项目.

以下文档发布在
[kubernetes.io/docs/reference](/docs/reference/):

* [Federation API v1 操作](https://kubernetes.io/docs/reference/federation/v1/operations/)
* [Federation API v1 定义](https://kubernetes.io/docs/reference/federation/v1/definitions/)
* [Federation API extensions/v1beta1 操作](https://kubernetes.io/docs/reference/federation/extensions/v1beta1/operations/)
* [Federation API extensions/v1beta1 定义](https://kubernetes.io/docs/reference/federation/extensions/v1beta1/definitions/)

{% endcapture %}

{% capture whatsnext %}



* [为Kubernetes API生成参考文档](/docs/home/contribute/generated-reference/kubernetes-api/)
* [为kubectl命令生成参考文档](/docs/home/contribute/generated-reference/kubectl/)
* [为Kubernetes组件和工具生成参考页面](/docs/home/contribute/generated-reference/kubernetes-components/)


{% endcapture %}


{% include templates/task.md %}
