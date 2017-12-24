---
cn-approvers:
- tianshapjq
approvers:
- bgrant0607
- thockin
title: Kubernetes 文档
---



Kubernetes 文档可以帮助您设置 Kubernetes，了解系统，或者让您的应用程序和工作负载在 Kubernetes 上运行。如果想要了解 Kubernetes 是什么，和它是如何运行的，请参阅 "[Kubernetes 是什么](/docs/concepts/overview/what-is-kubernetes/)"。


## 互动教程


[Kubernetes 基本概念的互动教程](/docs/tutorials/kubernetes-basics/) 通过一个虚拟的终端让您能够在网页浏览器中尝试 Kubernetes。您能够在几分钟内了解 Kubernetes 系统以及如何部署、暴露、伸缩和升级一个容器化的应用。


## 安装/设置 Kubernetes


无论是在本地开发环境还是在您选择的云提供商，[采用正确的方案](/docs/setup/pick-right-solution/) 都能够帮助您启动并运行一个 Kubernetes 集群。


## 概念、任务和教程


Kubernetes 文档包含了大量的资源，能够帮助您理解和使用 Kubernetes。


* [概念](/docs/concepts/) 提供了对 Kubernetes 如何工作的深入理解。
* [任务](/docs/tasks/) 包含对常见 Kubernetes 任务的分步说明。
* [教程](/docs/tutorials/) 包含 Kubernetes 工作流程的详细教程。


## API 和命令参考


[参考](/docs/reference/) 文档提供了 Kubernetes API 和 `kubectl` 命令行界面的完整信息。


## 工具


[工具](/docs/tools/) 页面包含 Kubernetes 的原生和第三方工具列表。


## 故障排查


[故障排查](/docs/tasks/debug-application-cluster/troubleshooting) 页面概述了一些用于故障排查和寻求帮助的资源。


## 支持的版本


Kubernetes 的版本以 _X.Y.Z_ 模式命名，其中 _X_ 是主版本号，_Y_ 是小版本号，_Z_ 是补丁版本号。

Kubernetes 一次支持三个小版本，也就是包含当前的发布版本和两个之前的版本。

参阅 GitHub 上的 [Kubernetes Release](https://github.com/kubernetes/kubernetes/releases) 页面以获取最新的发布信息。


### 小版本


master 组件、node 组件和 kubectl 客户端之间允许有一定的版本偏差。Node 可能落后 master 两个小版本，但是不能超过 master 版本。客户端可能落后也可能超过 master 一个小版本。


例如，预计一个 v1.8 的 master 将和 v1.6、v1.7 和 v1.8 的 node 兼容，并且和 v1.7、v1.8 和 v1.9 的客户端兼容。


### 补丁版本


补丁版本通常包含对关键错误的修复。您应该运行给定小版本的最新补丁版本。
