---
title: 安装 Minikube
cn-approvers:
- chentao1596
---



{% capture overview %}




{% endcapture %}

{% capture prerequisites %}


必须在您计算机的 BIOS 中启用 VT-x 或者 AMD-v 虚拟化。

{% endcapture %}

{% capture steps %}


## 安装 Hypervisor


如果尚未安装 hypervisor，请立即安装。


* 对于 OS X，请安装
[xhyve 驱动](https://git.k8s.io/minikube/docs/drivers.md#xhyve-driver)，
[VirtualBox](https://www.virtualbox.org/wiki/Downloads)，或者
[VMware Fusion](https://www.vmware.com/products/fusion)，或者
[HyperKit](https://github.com/moby/hyperkit)。


* 对于 Linux，请安装
[VirtualBox](https://www.virtualbox.org/wiki/Downloads) 或者
[KVM](http://www.linux-kvm.org/)。


* 对于 Windows，请安装
[VirtualBox](https://www.virtualbox.org/wiki/Downloads) 或者
[Hyper-V](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)。


## 安装 kubectl

* [安装 kubectl](/docs/tasks/tools/install-kubectl/)。


## 安装 Minikube

* 根据 [最新发行版](https://github.com/kubernetes/minikube/releases) 的说明安装 Minikube。

{% endcapture %}

{% capture whatsnext %}


* [通过 Minikube 在本地运行 Kubernetes](/docs/getting-started-guides/minikube/)

{% endcapture %}

{% include templates/task.md %}
