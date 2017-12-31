---
approvers:
- caesarxuchao
- erictune
title: oVirt
---

* TOC
{:toc}


## 什么是 oVirt


oVirt 是一个虚拟数据中心管理器，提供多宿主机上多虚拟机的强大管理能力。oVirt 使用 KVM 和 libvirt，可以在 Fedora、CentOS 或 Red Hat Enterprise Linux 主机上安装，以建立和管理您的虚拟数据中心。


## 部署 oVirt Cloud Provider


oVirt cloud provider 能够轻松的发现并自动将新的虚拟机实例添加为您的 Kubernetes 集群节点。目前还没有社区支持或预先加载的包含 Kubernetes 的虚拟机镜像，但是可以通过在虚拟机中 [导入] 或 [安装] Project Atomic（或者 Fedora）来 [生成模板]。任何包含 Kubernetes 的其它发行版同样可以运行。


必须在 guest 虚拟机中 [安装 ovirt-guest-agent]，以将虚拟机 IP 地址和主机名报告给 ovirt-engine 并最终报告给 Kubernetes。


一旦 Kubernetes 模板可用，就可以开始实例化可由 cloud provider 发现的虚拟机。


[导入]: http://ovedou.blogspot.it/2014/03/importing-glance-images-as-ovirt.html
[安装]: https://www.ovirt.org/documentation/quickstart/quickstart-guide/#create-virtual-machines
[生成模板]: https://www.ovirt.org/documentation/quickstart/quickstart-guide/#using-templates
[安装 ovirt-guest-agent]: http://www.ovirt.org/documentation/how-to/guest-agent/install-the-guest-agent-in-fedora/


## 使用 oVirt Cloud Provider


oVirt Cloud Provider 需要访问  oVirt REST-API 以收集适当的信息，所需的凭据应该在 `ovirt-cloud.conf` 文件中指定：

    [connection]
    uri = https://localhost:8443/ovirt-engine/api
    username = admin@internal
    password = admin


在同一个文件中，可以指定（使用 `filters` 部分）要将哪些用于识别虚拟机的搜索查询（search query）报告给 Kubernetes：

    [filters]
    # Search query used to find nodes
    vms = tag=kubernetes


在上面的例子中，所有标有 `kubernetes` 标签的虚拟机都会被当成节点报告给 Kubernetes。


必须在  kube-controller-manager 中指定 `ovirt-cloud.conf` 文件：

    kube-controller-manager ... --cloud-provider=ovirt --cloud-config=/path/to/ovirt-cloud.conf ...


## oVirt Cloud Provider 截屏


这个简短的截屏演示演示了如何使用oVirt Cloud Provider将虚拟机动态添加到您的Kubernetes集群。


[![截屏](http://img.youtube.com/vi/JyyST4ZKne8/0.jpg)](http://www.youtube.com/watch?v=JyyST4ZKne8)


##支持级别


IaaS Provider        | Config. Mgmt | OS     | Networking  | Docs                                              | Conforms | Support Level
-------------------- | ------------ | ------ | ----------  | ---------------------------------------------     | ---------| ----------------------------
oVirt                |              |        |             | [docs](/docs/getting-started-guides/ovirt)                                  |          | Community ([@simon3z](https://github.com/simon3z))


有关所有解决方案的支持级别信息，请参见[解决方案表](/docs/getting-started-guides/#table-of-solutions)。

