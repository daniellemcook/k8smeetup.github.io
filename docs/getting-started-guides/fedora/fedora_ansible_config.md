---
cn-approvers:
- tianshapjq
approvers:
- aveshagarwal
- erictune
title: 使用 Ansible 配置 Fedora
---



Ansible 提供了一种简单的方法在 Fedora 上配置 Kubernetes，能够快速创建一个集群环境。

* TOC
{:toc}


## 先决条件


1. 能够运行 ansible 并且能够克隆以下存储库的主机：[Kubernetes](https://github.com/kubernetes/kubernetes.git)
2. 一台至少 Fedora 21 的主机，用来作为集群的 master
3. 多个至少 Fedora 21 的主机，用来作为集群的 node

以上主机可以是虚拟机或者物理裸机。Ansible将负责为您完成其余的配置 - 配置网络、安装软件包和处理防火墙等。本示例将使用一个 master 节点和两个 node 节点。


## 集群架构

一个 Kubernetes 集群需要 etcd、master 和 n 个 node，因此我们将创建一个包含三台主机的集群，例如：

```shell
master,etcd = kube-master.example.com
    node1 = kube-node-01.example.com
    node2 = kube-node-02.example.com
```


**确保您的本地机器满足以下条件**

 - ansible (must be 1.9.0+)
 - git
 - python-netaddr


如果不满足

```shell
dnf install -y ansible git python-netaddr
```


**现在从 Kubernetes 存储库克隆文件**

```shell
git clone https://github.com/kubernetes/contrib.git
cd contrib/ansible
```


**配置集群中的每台机器及其角色**


获得 master 和 node 的 IP 地址。然后添加到运行 Ansible 的主机的 `~/contrib/ansible/inventory/localhost.ini` 文件。

```shell
[masters]
kube-master.example.com

[etcd]
kube-master.example.com

[nodes]
kube-node-01.example.com
kube-node-02.example.com
```


## 使 ansible 能访问您的所有 node


如果您的主机已经能够和 kube-master 节点及 kube-node-{01,02} 节点建立无需密码的 ssh 访问，并且有 'sudo' 权限，那么只需要将 ssh 访问的用户名（例如，`fedora`）配置到 `~/contrib/ansible/inventory/group_vars/all.yml` 文件的 `ansible_ssh_user` 中，然后继续下一步...


*否则* 需要需要您自己配置 ssh 以达到无需密码的 ssh 访问（您需要知道集群中所有机器的 root 密码）。


编辑：~/contrib/ansible/inventory/group_vars/all.yml

```yaml
ansible_ssh_user: root
```


**配置对集群的 ssh 访问**


如果您已经可以使用 ssh 公钥访问每台机器，那么您可以跳到 [配置群集](#setting-up-the-cluster)。

如果您的本地机器（root 用户）没有 ssh 的秘钥对，可以通过以下方式生成

```shell
ssh-keygen
```


拷贝这个 ssh 公钥到集群中的 **所有** 节点

```shell
for node in kube-master.example.com kube-node-01.example.com kube-node-02.example.com; do
  ssh-copy-id ${node}
done
```


## 配置集群


`~/contrib/ansible/inventory/group_vars/all.yml` 中的默认值应该已经足够了，如果不满足您的要求，您可以按照要求进行修改。

```conf
edit: ~/contrib/ansible/inventory/group_vars/all.yml
```


**配置到 Kubernetes 包的访问**

如下所示，修改 `source_type` 以能够通过包管理器访问 Kubernetes 包。

```yaml
source_type: packageManager
```


**配置服务所用的 IP 地址**

每一个 Kubernetes 服务都需要获得它自己的 IP 地址，这些并不是真实的 IP，您只需要选择一个在您的环境中未被使用的 IP 地址范围即可。

```yaml
kube_service_addresses: 10.254.0.0/16
```


**管理 flannel**

只有在默认值不适合您集群的情况下才需要修改 `flannel_subnet`、`flannel_prefix` 和 `flannel_host_prefix`。


**管理集群中的插件服务（add on services）**

将 `cluster_logging` 设置为 false 或 true（默认）以禁用或启用 elasticsearch 进行日志记录。

```yaml
cluster_logging: true
```


将 `cluster_monitoring` 设置为 true（默认）或 false 以启用或禁用使用 heapster 和 influxdb 进行集群监视。

```yaml
cluster_monitoring: true
```


将 `dns_setup` 设置为 true（推荐）或 false 以启用或禁用整个 DNS 配置。

```yaml
dns_setup: true
```


**叫 ansible 起来工作！**

这将最终为您建立整个 Kubernetes 群集。

```shell
cd ~/contrib/ansible/

./scripts/deploy-cluster.sh
```


## 测试并使用您的新集群

这就是创建集群的步骤，真的是非常简单。到这后您应该就拥有一个可以工作的 Kubernetes 集群了。


**展示 Kubernetes nodes**

在 kube-master 上运行如下命令：

```shell
kubectl get nodes
```


**展示运行在 master 和 node 上的服务**

```shell
systemctl | grep -i kube
```


**展示在 master 和 node 上的防火墙规则**

```shell
iptables -nvL
```


**使用以下内容在 master 上创建 /tmp/apache.json 并部署 pod**

```json
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "fedoraapache",
    "labels": {
      "name": "fedoraapache"
    }
  },
  "spec": {
    "containers": [
      {
        "name": "fedoraapache",
        "image": "fedora/apache",
        "ports": [
          {
            "hostPort": 80,
            "containerPort": 80
          }
        ]
      }
    ]
  }
}
```

```shell
kubectl create -f /tmp/apache.json
```


**检查 pod 创建的位置**

```shell
kubectl get pods
```


**检查 node 上的 Docker 状态**

```shell
docker ps
docker images
```


**在 pod 处于 'Running' 状态后，通过在 node 上访问以检查 web 服务**

```shell
curl http://localhost
```


就是这么简单！


## 支持级别


IaaS 提供者          | Config. Mgmt | 系统   | 网络        | 文档                                              | 配套     | 支持级别
-------------------- | ------------ | ------ | ----------  | ---------------------------------------------     | ---------| ----------------------------
裸机                 | Ansible      | Fedora | flannel     | [docs](/docs/getting-started-guides/fedora/fedora_ansible_config)           |          | Project


有关所有解决方案的支持级别信息，请参见 [解决方案表](/docs/getting-started-guides/#table-of-solutions)。
