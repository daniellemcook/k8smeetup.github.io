---
approvers:
- david-mcmahon
- jbeda
title: 从源代码构建
---



您可以从源代码构建版本，也可以下载预构建版本。 如果您不打算对 Kubernetes 本身进行开发，
我们建议使用当前版本的预构建版本，预构建版本可以在 [版本说明](/docs/imported/release/notes/) 里找到。


Kubernetes 源代码可以从 [kubernetes/kubernetes 仓库](https://github.com/kubernetes/kubernetes) 下载。


### 从源代码构建


如果您只是从源代码构建一个版本，那么就不必设置完整的 golang 环境，因为整个构建都发生在 Docker 容器中。


构建一个版本很简单。

```shell
git clone https://github.com/kubernetes/kubernetes.git
cd kubernetes
make release
```


更多版本构建流程相关的详情，请查看 kubernetes/kubernetes [`build`](http://releases.k8s.io/{{page.githubbranch}}/build/) 目录。
