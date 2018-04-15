---
cn-approvers:
- zhangqx2010
title: 在 CenturyLink Cloud 上运行 Kubernetes
---


* TOC
{: toc}


这些脚本处理 CenturyLink Cloud 上 Kubernetes 集群的创建，删除和扩展。


您可以使用单个命令完成所有这些任务。我们已经可以使用 Ansible playbooks 执行这些任务，请参考 [这里](https://github.com/CenturyLinkCloud/adm-kubernetes-on-clc/blob/master/ansible/README.md)。


## 寻求帮助


如果遇到任何问题或需要任何帮助，我们都会帮助您。您可以通过以下任何方式联系我们：

- 提交一个 github 问题
- 发送电子邮件给 Kubernetes@ctl.io
- 访问 [http://info.ctl.io/kubernetes](http://info.ctl.io/kubernetes)


## 您可以任意选择虚拟机或物理服务器组成集群


- 无论虚拟机还是物理服务器均支持 Kubernetes 集群。如果工作节点（minion）使用物理服务器，请使用 --minion_type=bareMetal 标志。
- 有关物理服务器的更多信息，请访问：[https://www.ctl.io/bare-metal/](https://www.ctl.io/bare-metal/)
- 物理服务仅在 VA1 和 GB3 数据中心提供。
- 所有13个公有云位置均提供虚拟机


## 要求



运行这个脚本的要求是：
- 一个 Linux 管理主机（在 Ubuntu 和 OSX 测试过）
- python 2（在 2.7.11 上测试过）
  - pip（安装 Python 2.7.9）
- git
- 具有创建新主机权限的 CenturyLink Cloud 帐户
- 从您的 Linux 主机到 CenturyLink Cloud 的 VPN 连接


## 脚本安装


满足所有要求后，请按照以下说明运行安装脚本。


1) 克隆这个仓库，并 cd 到这个目录中。

```shell
git clone https://github.com/CenturyLinkCloud/adm-kubernetes-on-clc
```


2) 安装所有要求，包括

  * Ansible
  * CenturyLink Cloud SDK
  * Ansible模块

```shell
sudo pip install -r ansible/requirements.txt
```


3) 从模板创建凭证文件，并使用它来设置您的ENV变量

```shell
cp ansible/credentials.sh.template ansible/credentials.sh
vi ansible/credentials.sh
source ansible/credentials.sh

```


4) 使用网络内的虚拟机或 [配置连接到 CenturyLink Cloud 网络的 VPN](https://www.ctl.io/knowledge-base/network/how-to-configure-client-vpn/) 来从您的机器访问 CenturyLink Cloud 网络。


#### 脚本安装示例：Ubuntu 14 演练


如果您使用的是 Ubuntu 14，方便起见，我们已经提供了一个安装依赖和脚本的分步指导。

```shell
# system
apt-get update
apt-get install -y git python python-crypto
curl -O https://bootstrap.pypa.io/get-pip.py
python get-pip.py

# installing this repository
mkdir -p ~home/k8s-on-clc
cd ~home/k8s-on-clc
git clone https://github.com/CenturyLinkCloud/adm-kubernetes-on-clc.git
cd adm-kubernetes-on-clc/
pip install -r requirements.txt

# getting started
cd ansible
cp credentials.sh.template credentials.sh; vi credentials.sh
source credentials.sh
```




## 集群创建


要创建一个新的 Kubernetes 集群，只需运行 ```kube-up.sh``` 脚本。以下列出了脚本选项和一些示例的完整列表。

```shell
CLC_CLUSTER_NAME=[name of kubernetes cluster]
cd ./adm-kubernetes-on-clc
bash kube-up.sh -c="$CLC_CLUSTER_NAME"
```


大约需要15分钟来创建集群。一旦脚本完成，它会输出一些命令来帮助您在机器上设置 kubectl 以指向新的集群。


集群创建完成后，会将其配置文件本地保存在您管理主机的以下目录中

```shell
> CLC_CLUSTER_HOME=$HOME/.clc_kube/$CLC_CLUSTER_NAME/
```



#### 集群创建：脚本选项

```shell
Usage: kube-up.sh [OPTIONS]
Create servers in the CenturyLinkCloud environment and initialize a Kubernetes cluster
Environment variables CLC_V2_API_USERNAME and CLC_V2_API_PASSWD must be set in
order to access the CenturyLinkCloud API

All options (both short and long form) require arguments, and must include "="
between option name and option value.

     -h (--help)                   display this help and exit
     -c= (--clc_cluster_name=)     set the name of the cluster, as used in CLC group names
     -t= (--minion_type=)          standard -> VM (default), bareMetal -> physical]
     -d= (--datacenter=)           VA1 (default)
     -m= (--minion_count=)         number of kubernetes minion nodes
     -mem= (--vm_memory=)          number of GB ram for each minion
     -cpu= (--vm_cpu=)             number of virtual cps for each minion node
     -phyid= (--server_conf_id=)   physical server configuration id, one of
                                      physical_server_20_core_conf_id
                                      physical_server_12_core_conf_id
                                      physical_server_4_core_conf_id (default)
     -etcd_separate_cluster=yes    create a separate cluster of three etcd nodes,
                                   otherwise run etcd on the master node
```


## 集群扩展


要扩展现有的 Kubernetes 集群，运行 ```add-kube-node.sh``` 脚本。脚本选项和一些例子的完整列表在 [下面](#cluster-expansion-script-options) 列出。
此脚本必须从创建集群的同一台主机（或者在 ~/.clc_kube/$cluster_name 路径保存了集群 artifact 文件的主机）运行。

```shell
cd ./adm-kubernetes-on-clc
bash add-kube-node.sh -c="name_of_kubernetes_cluster" -m=2
```


#### 集群扩展：脚本选项

```shell
Usage: add-kube-node.sh [OPTIONS]
Create servers in the CenturyLinkCloud environment and add to an
existing CLC kubernetes cluster

Environment variables CLC_V2_API_USERNAME and CLC_V2_API_PASSWD must be set in
order to access the CenturyLinkCloud API

     -h (--help)                   display this help and exit
     -c= (--clc_cluster_name=)     set the name of the cluster, as used in CLC group names
     -m= (--minion_count=)         number of kubernetes minion nodes to add
```


## 集群删除


有两种方法可以删除现有的集群：


1) 使用我们的 Python 脚本：

```shell
python delete_cluster.py --cluster=clc_cluster_name --datacenter=DC1
```


2) 使用 CenturyLink Cloud 用户界面。
要删除集群，请登录到 CenturyLink Cloud 控制门户并删除包含该的父服务器组 Kubernetes 集群。
我们希望添加一个脚本化的选项来尽快做到这一点。


## 例子


在 VA1 中创建名称为 k8s_1，1个 master 节点和3个 worker（在物理机上）的集群

```shell
bash kube-up.sh --clc_cluster_name=k8s_1 --minion_type=bareMetal --minion_count=3 --datacenter=VA1
```


在 VA1 中创建一个名为 k8s_2 的集群，一个在3个虚拟机和6个 worker minion（在虚拟机上）的 HA etcd 集群

```shell
bash kube-up.sh --clc_cluster_name=k8s_2 --minion_type=standard --minion_count=6 --datacenter=VA1 --etcd_separate_cluster=yes
```


在 UC1 中创建一个名为 k8s_3，1个 master 节点和10个具有更高 mem/cpu 的 worker minion（在VM上）的集群：

```shell
bash kube-up.sh --clc_cluster_name=k8s_3 --minion_type=standard --minion_count=10 --datacenter=VA1 -mem=6 -cpu=4
```




## 集群功能和体系结构


我们使用以下功能配置 Kubernetes 集群：


* KubeDNS：DNS 解析和服务发现
* Heapster/InfluxDB：用于 metric 收集。需要 Grafana 和 auto-scaling。
* Grafana：Kubernetes/Docker metric 面板
* KubeUI：简单的界面来查看 Kubernetes 状态
* Kube Dashboard：新的用于与您的集群交互的 Web 界面


我们使用这些组件创建 Kubernetes 集群：

* Kubernetes 1.1.7
* Ubuntu 14.04
* Flannel 0.5.4
* Docker 1.9.1-0~trusty
* Etcd 2.2.2


## 可选插件


* 日志记录：我们提供一个集成的集中日志记录 ELK 平台，以便所有 Kubernetes 和 docker 日志被发送到 ELK 堆栈。要安装 ELK 堆栈并配置 Kubernetes 发送日志，
  遵循 [日志聚合文档](https://github.com/CenturyLinkCloud/adm-kubernetes-on-clc/blob/master/log_aggregration.md)。
  注意：我们不会默认安装这个，因为它占用的空间并不小。


## 集群管理


管理 Kubernetes 集群的最广泛使用的工具是命令行 ```kubectl```。
如果您的管理主机上还没有这个二进制文件，可以运行 ```install_kubectl.sh```，这个脚本会将它下载并安装到 ```/usr/bin/local``` 中。


该脚本要求定义环境变量 ```CLC_CLUSTER_NAME```


```install_kubectl.sh``` 也写一个配置文件，它将嵌入必要的特定集群的身份验证证书。
配置文件会写到 ```${CLC_CLUSTER_HOME}/kube``` 目录

```shell
export KUBECONFIG=${CLC_CLUSTER_HOME}/kube/config
kubectl version
kubectl cluster-info
```


### 以编程方式访问集群


可以使用本地存储的客户端证书来访问 apiserver。例如，您可能希望使用任何 [Kubernetes API 客户端库](/docs/reference/client-libraries/) ，并以您选择的编程语言对您的 Kubernetes 集群进行编程。


为了演示如何使用这些本地存储的证书，我们提供了以下例子，使用 ```curl`` 通过 https 与主 apiserver 进行通信：

```shell
curl \
   --cacert ${CLC_CLUSTER_HOME}/pki/ca.crt  \
   --key ${CLC_CLUSTER_HOME}/pki/kubecfg.key \
   --cert ${CLC_CLUSTER_HOME}/pki/kubecfg.crt  https://${MASTER_IP}:6443
```


但请注意，这*不*适用于 OSX 版本的 curl 二进制文件。


### 使用浏览器访问集群


我们在 Kubernetes 上安装两个 UI。原来的 KubeUI 和 [新的 Kube dashboard](/docs/tasks/web-ui-dashboard/)。
当你创建一个集群时，脚本应该输出这些接口 URL ，像这样：

KubeUI is running at ```https://${MASTER_IP}:6443/api/v1/namespaces/kube-system/services/kube-ui/proxy```.

kubernetes-dashboard is running at ```https://${MASTER_IP}:6443/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy```.


关于用户界面身份验证的注意事项：集群设置为使用基本用户 _admin_ 的身份验证。打在网址 ```https://${MASTER_IP}:6443``` 需要接受 apiserver 的自签名证书，然后提交管理员密码文件，位于：

```> _${CLC_CLUSTER_HOME}/kube/admin_password.txt_```



### 配置文件


各种配置文件被写入主目录 *CLC_CLUSTER_HOME* 下的 ```.clc_kube/${CLC_CLUSTER_NAME}``` 几个子目录中。
你可以使用这些文件从您创建集群的位置以外的机器访问集群。


* ```config/```: Ansible变量文件，包含描述主控和主控主机的参数
* ```hosts/```: 主机文件列出了可靠的剧本的访问信息
* ```kube/```: ```kubectl``` 配置文件，以及管理员访问 Kubernetes API 的基本认证密码
* ```pki/```: 在集群中启用 TLS 通信的公共基础设施文件
* ```ssh/```: 用于 root 访问主机的 SSH 密钥


## ```kubectl``` 使用例子


_kubectl_ 有很多功能。下面是一些例子


列出所有名称空间中的现有 node、Pod、service 等等，或者只列出一个：

```shell
kubectl get nodes
kubectl get --all-namespaces services
kubectl get --namespace=kube-system replicationcontrollers
```


Kubernetes API 服务器在 web URL 上提供服务，通过校验客户证书保护这些 URL 。
如果你在本地运行一个 kubectl 代理，```kubectl``` 将会提供必要的证书并通过 http 服务于本地。

```shell
kubectl proxy -p 8001
```


然后，您可以在浏览器中访问像 ```http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/kube-ui/proxy/``` 这样的 URL 而不需要客户端证书。



## 哪些 Kubernetes 特性在 CenturyLink Cloud 上不起作用


这些是在 CenturyLink Cloud 上不起作用的已知项，但可以在其他云上工作：


- 目前没有 [LoadBalancer](/docs/tasks/access-application-cluster/create-external-load-balancer/) 类型的支持服务。我们正在积极努力，希望在2016年4月左右发布这些变化。

- 目前，CenturyLink Cloud 不支持持久性存储卷。但是，客户可以使用自己选中的持久存储。我们自己使用 Gluster。


## Ansible 文件


如果你想了解更多有关我们 Ansible 文件的信息，请 [阅读此文件](https://github.com/CenturyLinkCloud/adm-kubernetes-on-clc/blob/master/ansible/README.md)


## 更多信息

有关管理和使用 Kubernetes 集群的更多详细信息，请参阅 [Kubernetes 文档](/docs/)。
