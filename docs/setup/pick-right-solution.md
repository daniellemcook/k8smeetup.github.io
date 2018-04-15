---
approvers:
- brendandburns
- erictune
- mikedanese
title: 选择正确的解决方案
cn-approvers:
- chentao1596
---



Kubernetes 可以在各种平台上运行：从您的笔记本电脑到云服务提供商的 VM，再到一架裸机服务器。设置群集所需的工作量从运行单个命令到自己的定制集群都不一样。使用本指南可选择适合您需要的解决方案。


如果您只是想在 Kubernetes 上 “浅尝辄止”，请参考 [使用 MiniKube 在本地基于 Docker 的解决方案](#本地机器解决方案)。


当您准备扩展到更多的机器以及更高的可用性时，[托管解决方案](#托管解决方案) 是最容易创建和维护的。


[Turnkey 云解决方案](#turnkey-云解决方案) 只需要几个命令就可以创建和覆盖广泛的云服务提供商。[本地 turnkey 云解决方案](#本地 turnkey 云解决方案) 与您自己私有网络的安全性相结合，具有 turnkey 云解决方案的简单性。


如果您已经有了一种配置宿主机资源的方法，那么，每台机器只需一个命令，就可以使用 [kubeadm](/docs/setup/independent/create-cluster-kubeadm/) 轻松地创建一个集群。


[自定义解决方案](#自定义解决方案) 从一步一步的指示到一般建议，从零开始设置 Kubernetes 集群。

* TOC
{:toc}


# 本地机器解决方案


* [Minikube](/docs/getting-started-guides/minikube/) 是为开发和测试创建本地的、单节点的 Kubernetes 集群的推荐方法。安装是完全自动化的，不需要云服务提供商的帐户。


* [Kubeadm-dind](https://github.com/Mirantis/kubeadm-dind-cluster) 是一个多节点（而 minkube 是单节点的）的 Kubernetes 集群，它只需要一个 docker 守护进程。它使用 docker-in-docker 技术来创建 Kubernetes 集群。


* [LXD 上的 Ubuntu](/docs/getting-started-guides/ubuntu/local/) 支持在本地主机上部署 9 个实例。


* [IBM Cloud Private-CE （社区版）](https://github.com/IBM/deploy-ibm-cloud-private) 可以在您的计算机上使用 VirtualBox 将 Kubernetes 部署到一个或多个 VM 中，用于开发和测试场景。可以扩展为完整的多节点集群。


# 托管解决方案


* [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) 提供托管 Kubernetes 集群。


* [Azure Container Service](https://azure.microsoft.com/en-us/services/container-service/) 可以轻松地部署 Kubernetes 集群。


* [Stackpoint.io](https://stackpoint.io) 为多个公有云提供 Kubernetes 基础设施自动化和管理


* [AppsCode.com](https://appscode.com/products/cloud-deployment/) 为各种公有云提供托管 Kubernetes 集群，包括 AWS 和 Google 云平台


* [KUBE2GO.io](https://kube2go.io) 开始使用多个公有云上的高可用 Kubernetes 集群，以及用于开发、调试和监视的有用工具。


* [Madcore.Ai](https://madcore.ai) 是专注于 devops
（devops-focused）的 CLI 工具，用于在 AWS 中部署 Kubernetes 基础设施。包括：Master、自动缩放带有实例的组节点、ingress-ssl-lego、Heapster 以及 Grafana。


* [Platform9](https://platform9.com/products/kubernetes/) 在本地或者任何公有云上提供托管 Kubernetes，并提供 24/7 健康监测和警报。


* [OpenShift Dedicated](https://www.openshift.com/dedicated/) 提供由 OpenShift 驱动的托管 Kubernetes 集群。


* [OpenShift Online](https://www.openshift.com/features/) 为 Kubernetes 应用提供免费托管访问。


* [IBM Cloud Container Service](https://console.bluemix.net/docs/containers/container_index.html) 提供了托管 Kubernetes 集群，包括隔离选择、操作工具、对镜像和容器的集成安全洞察以及与 Watson、物联网和数据的集成。


* [Giant Swarm](https://giantswarm.io/product/) 在自己的数据中心、本地或公有云上提供托管 Kubernetes 集群。


# Turnkey 云解决方案


通过这些解决方案，您可以只使用几条命令就可以在一系列云 IaaS 提供商上创建 Kubernetes 集群。这些解决方案得到了积极的发展，并得到了的社区的积极支持。


* [在 AWS、Azure、Google 云、Oracle 云上使用 Ubuntu 搭建 Kubernetes](/docs/getting-started-guides/ubuntu/)
* [Google Compute Engine (GCE)](/docs/getting-started-guides/gce/)
* [AWS](/docs/getting-started-guides/aws/)
* [Azure](/docs/getting-started-guides/azure/)
* [使用 CoreOS 的 Tectonic](https://coreos.com/tectonic)
* [CenturyLink 云](/docs/getting-started-guides/clc/)
* [IBM 云](https://github.com/patrocinio/kubernetes-softlayer)
* [Stackpoint.io](/docs/getting-started-guides/stackpoint/)
* [KUBE2GO.io](https://kube2go.io/)
* [Madcore.Ai](https://madcore.ai/)


# 本地 turnkey 云解决方案

这些解决方案允许您在内部、安全的云网络上创建 Kubernetes 集群，只需几个命令即可。


* [IBM 私有云](https://www.ibm.com/cloud-computing/products/ibm-cloud-private/)


# 自定义解决方案


Kubernetes 可以在广泛的云服务提供商和裸机环境中运行，并且支持许多基本的操作系统。


如果能找到符合您需要的指南，就请使用它吧。因为，虽然它可能有点过时，但总比从头开始来的容易。如果您确实希望从头开始，或者因为您有特殊的需求，又或者仅仅因为您想了解 Kubernetes 集群下面的内容，那么就请尝试 [从头开始](/docs/getting-started-guides/scratch/) 指南。


如果您对在新平台上支持 Kubernetes 感兴趣，请参见 [编写入门指南](https://git.k8s.io/community/contributors/devel/writing-a-getting-started-guide.md)。


## 普遍的


如果您已经有了一种配置宿主机资源的方法，那么，每台机器只需一个命令，就可以使用 [kubeadm](/docs/setup/independent/create-cluster-kubeadm/) 轻松地创建一个集群。


## 云


这些解决方案是上面的解决方案没有涵盖的云服务提供商和操作系统的组合。


* [AWS 或 GCE 上的 CoreOS](/docs/getting-started-guides/coreos/)
* [Ubuntu 上的 Kubernetes](/docs/getting-started-guides/ubuntu/)
* [Kubespray](/docs/getting-started-guides/kubespray/)


## 本地 VM（虚拟机）


* [Vagrant](/docs/getting-started-guides/coreos/)（使用 CoreOS 和 flannel）
* [CloudStack](/docs/getting-started-guides/cloudstack/)（使用 Ansible、CoreOS 和 flannel）
* [Vmware vSphere](/docs/getting-started-guides/vsphere/)（使用 Debian）
* [Vmware Photon Controller](/docs/getting-started-guides/photon-controller/)（使用 Debian）
* [Vmware vSphere、OpenStack 或者裸机](/docs/getting-started-guides/ubuntu/)（使用 Juju、Ubuntu 和 flannel）
* [Vmware](/docs/getting-started-guides/coreos/)（使用 CoreOS 和 flannel）
* [libvirt 上的 CoreOS](/docs/getting-started-guides/libvirt-coreos/)（使用 CoreOS）
* [oVirt](/docs/getting-started-guides/ovirt/)
* [OpenStack Heat](/docs/getting-started-guides/openstack-heat/)（使用 CentOS 和 flannel）
* [Fedora（多节点）](/docs/getting-started-guides/fedora/flannel_multi_node_cluster/)（使用 Fedora 和 flannel）


## 裸机


* [离线](/docs/getting-started-guides/coreos/bare_metal_offline/)（无需上网，使用 CoreOS 和 Flannel）
* [Fedora 使用 Ansible](/docs/getting-started-guides/fedora/fedora_ansible_config/)
* [Fedora（单节点）](/docs/getting-started-guides/fedora/fedora_manual_config/)
* [Fedora（多节点）](/docs/getting-started-guides/fedora/flannel_multi_node_cluster/)
* [CentOS](/docs/getting-started-guides/centos/centos_manual_config/)
* [Ubuntu 上的 Kubernetes](/docs/getting-started-guides/ubuntu/)
* [AWS 或 GCE 上的 CoreOS](/docs/getting-started-guides/coreos/)


## 集成


这些解决方案提供与第三方调度程序、资源管理器和/或更低级别平台的集成。


* [Mesos 上的 Kubernetes](/docs/getting-started-guides/mesos/)
  * 操作指南指定 GCE, 但是它已经足够通用，完全可以适应大多数现有的 Mesos 集群。
* [DCOS](/docs/getting-started-guides/dcos/)
  * 社区版使用 AWS
  * 商业版支持云托管、本地虚拟机和裸机


# 解决方案表


下面是上面列出的所有解决方案的表格。


IaaS 提供商          | 配置管理     | OS     | 网络        | 文档                                              | 支持级别
-------------------- | ------------ | ------ | ----------  | ---------------------------------------------     | ----------------------------
任何                  | 任何          | 多操作系统支持 | 任何 CNI | [文档](/docs/setup/independent/create-cluster-kubeadm/) | 项目 ([SIG-cluster-lifecycle](https://git.k8s.io/community/sig-cluster-lifecycle))
Google Kubernetes 引擎 |              |        | GCE         | [文档](https://cloud.google.com/kubernetes-engine/docs/) | 商业
Stackpoint.io        |              | 多操作系统支持       | 多网络支持   | [文档](https://stackpoint.io/) | 商业
AppsCode.com         | Saltstack    | Debian | 多网络支持 | [文档](https://appscode.com/products/cloud-deployment/) | 商业
KUBE2GO.io          |              | 多操作系统支持 | 多网络支持 | [文档](https://kube2go.io) | 商业
Madcore.Ai           | Jenkins DSL  | Ubuntu | flannel     | [文档](https://madcore.ai)                        | 社区 ([@madcore-ai](https://github.com/madcore-ai))
Platform9        |              | 多操作系统支持 | 多网络支持 | [文档](https://platform9.com/managed-kubernetes/) | 商业
Giant Swarm        |              | CoreOS | flannel 和/或 Calico | [文档](https://docs.giantswarm.io/) | 商业
GCE                  | Saltstack    | Debian | GCE         | [文档](/docs/getting-started-guides/gce/)                                    | 项目
Azure Container Service |              | Ubuntu | Azure       | [文档](https://azure.microsoft.com/en-us/services/container-service/)                    |  商业
Azure (IaaS)    |              | Ubuntu | Azure       | [文档](/docs/getting-started-guides/azure/)                    |  [社区 (Microsoft)](https://github.com/Azure/acs-engine)
裸机         | Ansible      | Fedora | flannel     | [文档](/docs/getting-started-guides/fedora/fedora_ansible_config/)           |  项目
裸机           | 自定义       | Fedora | 无      | [文档](/docs/getting-started-guides/fedora/fedora_manual_config/)            |  项目
裸机           | 自定义       | Fedora | flannel     | [文档](/docs/getting-started-guides/fedora/flannel_multi_node_cluster/)      |  社区 ([@aveshagarwal](https://github.com/aveshagarwal))
libvirt              | 自定义       | Fedora | flannel     | [文档](/docs/getting-started-guides/fedora/flannel_multi_node_cluster/)      |  社区 ([@aveshagarwal](https://github.com/aveshagarwal))
KVM                  | 自定义       | Fedora | flannel     | [文档](/docs/getting-started-guides/fedora/flannel_multi_node_cluster/)      |  社区 ([@aveshagarwal](https://github.com/aveshagarwal))
Mesos/Docker         | 自定义       | Ubuntu | Docker      | [文档](/docs/getting-started-guides/mesos-docker/)                           |  社区 ([Kubernetes-Mesos Authors](https://github.com/mesosphere/kubernetes-mesos/blob/master/AUTHORS.md))
Mesos/GCE            |              |        |             | [文档](/docs/getting-started-guides/mesos/)                                  |  社区 ([Kubernetes-Mesos Authors](https://github.com/mesosphere/kubernetes-mesos/blob/master/AUTHORS.md))
DCOS                 | Marathon   | CoreOS/Alpine | 自定义 | [文档](/docs/getting-started-guides/dcos/)                                   |  社区 ([Kubernetes-Mesos Authors](https://github.com/mesosphere/kubernetes-mesos/blob/master/AUTHORS.md))
AWS                  | CoreOS       | CoreOS | flannel     | [文档](/docs/getting-started-guides/aws/)                                 |  社区
GCE                  | CoreOS       | CoreOS | flannel     | [文档](/docs/getting-started-guides/coreos/)                                 |  社区 ([@pires](https://github.com/pires))
Vagrant              | CoreOS       | CoreOS | flannel     | [文档](/docs/getting-started-guides/coreos/)                                 |  社区 ([@pires](https://github.com/pires), [@AntonioMeireles](https://github.com/AntonioMeireles))
裸机  (Offline) | CoreOS       | CoreOS | flannel     | [文档](/docs/getting-started-guides/coreos/bare_metal_offline/)              |  社区 ([@jeffbean](https://github.com/jeffbean))
CloudStack           | Ansible      | CoreOS | flannel     | [文档](/docs/getting-started-guides/cloudstack/)                             |  社区 ([@sebgoa](https://github.com/sebgoa))
Vmware vSphere       | Saltstack    | Debian | OVS         | [文档](/docs/getting-started-guides/vsphere/)                                |  社区 ([@imkin](https://github.com/imkin))
Vmware Photon        | Saltstack    | Debian | OVS         | [文档](/docs/getting-started-guides/photon-controller/)                      |  社区 ([@alainroy](https://github.com/alainroy))
裸机          | 自定义       | CentOS | flannel      | [文档](/docs/getting-started-guides/centos/centos_manual_config/)            |  社区 ([@coolsvap](https://github.com/coolsvap))
lxd                  | Juju         | Ubuntu | flannel/canal            | [文档](/docs/getting-started-guides/ubuntu/local/)              |  [商业](https://www.ubuntu.com/kubernetes) and [Community](https://jujucharms.com/kubernetes)
AWS                  | Juju         | Ubuntu | flannel/calico/canal     | [文档](/docs/getting-started-guides/ubuntu/)                    |  [商业](https://www.ubuntu.com/kubernetes) and [Community](https://jujucharms.com/kubernetes)
Azure                | Juju         | Ubuntu | flannel/calico/canal     | [文档](/docs/getting-started-guides/ubuntu/)                    |  [商业](https://www.ubuntu.com/kubernetes) and [Community](https://jujucharms.com/kubernetes)
GCE                  | Juju         | Ubuntu | flannel/calico/canal     | [文档](/docs/getting-started-guides/ubuntu/)                    |  [商业](https://www.ubuntu.com/kubernetes) and [Community](https://jujucharms.com/kubernetes)
Oracle 云         | Juju         | Ubuntu | flannel/calico/canal     | [文档](/docs/getting-started-guides/ubuntu/)                    |  [商业](https://www.ubuntu.com/kubernetes) and [Community](https://jujucharms.com/kubernetes)
Rackspace            | Juju         | Ubuntu | flannel/calico/canal     | [文档](/docs/getting-started-guides/ubuntu/)                    |  [商业](https://www.ubuntu.com/kubernetes) and [Community](https://jujucharms.com/kubernetes)
Vmware vSphere       | Juju         | Ubuntu | flannel/calico/canal     | [文档](/docs/getting-started-guides/ubuntu/)                    |  [商业](https://www.ubuntu.com/kubernetes) and [Community](https://jujucharms.com/kubernetes)
裸机           | Juju         | Ubuntu | flannel/calico/canal     | [文档](/docs/getting-started-guides/ubuntu/)                    |  [商业](https://www.ubuntu.com/kubernetes) and [Community](https://jujucharms.com/kubernetes)
AWS                  | Saltstack    | Debian | AWS         | [文档](/docs/getting-started-guides/aws/)                                    |  社区 ([@justinsb](https://github.com/justinsb))
AWS                  | kops         | Debian | AWS         | [文档](https://github.com/kubernetes/kops/)                                  |  社区 ([@justinsb](https://github.com/justinsb))
裸机           | 自定义       | Ubuntu | flannel     | [文档](/docs/getting-started-guides/ubuntu/)                                 |  社区 ([@resouer](https://github.com/resouer), [@WIZARD-CXY](https://github.com/WIZARD-CXY))
libvirt/KVM          | CoreOS       | CoreOS | libvirt/KVM | [文档](/docs/getting-started-guides/libvirt-coreos/)                         |  社区 ([@lhuard1A](https://github.com/lhuard1A))
oVirt                |              |        |             | [文档](/docs/getting-started-guides/ovirt/)                                  |  社区 ([@simon3z](https://github.com/simon3z))
OpenStack Heat       | Saltstack    | CentOS | Neutron + flannel hostgw | [文档](/docs/getting-started-guides/openstack-heat/)            |  社区 ([@FujitsuEnablingSoftwareTechnologyGmbH](https://github.com/FujitsuEnablingSoftwareTechnologyGmbH))
任何                  | 任何            |任何      | 任何           | [文档](/docs/getting-started-guides/scratch/)                                |  社区 ([@erictune](https://github.com/erictune))
任何                 | 任何            | 任何      | 任何           | [文档](http://docs.projectcalico.org/v2.2/getting-started/kubernetes/installation/)                                |  商业和社区


**注**：上面的表由节点中测试/使用的版本排序，其次是支持级别。


## 列定义


* **IaaS 提供商** 是提供 Kubernetes 运行的虚拟机或物理机器（节点）的产品或组织。

* **OS** 是节点的基本操作系统。

* **配置管理** 是帮助在节点上安装和维护 Kubernetes 的配置管理系统。

* **网络** 是实现 [网络模型](/docs/concepts/cluster-administration/networking/) 的工具。那些网络类型为 _none_ 的节点可能不支持多个节点，也可能支持单个物理节点中的多个 VM 节点。

* **一致性** 表示使用此配置创建的集群是否通过了项目的一致性测试，以支持 Kubernetes v1.0.0 的 API 和基本特性。

* **支持级别**
  * **项目**: Kubernetes 提交者经常使用此配置，因此它通常与 Kubernetes 的最新版本一起工作。
  * **商业**: 有自己的支持安排的商业产品。
  * **社区**: 积极支持社区贡献。可能对 Kubernetes 的最新版本不起作用。
  * **无效**: 未能积极维护的。不推荐给首次使用 Kubernetes 的用户，并且可能会被删除。


* **注** 包括其他有关资料，例如使用的 Kubernetes 版本。



[1]: https://gist.github.com/erictune/4cabc010906afbcc5061

[2]: https://gist.github.com/derekwaynecarr/505e56036cdf010bf6b6

[3]: https://gist.github.com/erictune/2f39b22f72565365e59b
