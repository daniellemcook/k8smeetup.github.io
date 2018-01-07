---
approvers:
- erictune
- idvoretskyi
cn-approvers:
- xiaosuiba
cn-reviewers:
- tianshapjq
title: 基于 libvirt 使用 CoreOS
---


* TOC
{:toc}


### 高亮特性

* 集群急速启动（秒级启动，而 vagrant 需要几分钟）。
* 降低磁盘使用率，这得益于 [COW](https://en.wikibooks.org/wiki/QEMU/Images#Copy_on_write)。
* 减少内存占用，这得益于 [KSM](https://www.kernel.org/doc/Documentation/vm/ksm.txt)


### 有关 `libvirt-coreos` 用例的警告


`libvirt-coreos` 集群提供程序的主要目的是尽可能快地在本地虚拟机上部署多节点的 Kubernetes 集群，并尽可能少的使用资源。


为了实现这一目标，其部署方式与其他提供商所使用的“标准生产部署”方法有很大不同。这样做的目的是为了实现一些优化，因为我们知道所有的虚拟机将运行在同一台物理机器上。


`libvirt-coreos` 集群提供程序的目标并不是生产环境。


另一个不同之处在于，`libvirt-coreos` 完全没有应用任何安全特性。例如，

* 通过明文连接（无 SSL）访问 Kube API Server；
* Kube API Server不需要凭证；
* etcd 访问不受保护；
* Kubernetes secret 没有像生产环境上那样进行保护；
* 等等。


因此，应用开发者不应该在 `libvirt-coreos` 验证其程序和 Kubernetes 的交互，因为他可能会在技术上成功地做到一些生产环境所禁止的事情，比如：

* 未经认证的访问 Kube API 服务器;
* 访问 etcd 中的 Kubernetes 私有数据结构；
* 等等。


另一方面，`libvirt-coreos` 可能对研究 Kubernetes 底层实现的人有用，因为像网络流量嗅探或 etcd 内容探测等调试技术在 `libvirt-coreos` 上比在生产环境上更容易实现。


### 准备工作

1. 安装 [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html)
1. 安装 [ebtables](http://ebtables.netfilter.org/)
1. 安装 [qemu](http://wiki.qemu.org/Main_Page)
1. 安装 [libvirt](http://libvirt.org/)
1. 安装  [openssl](http://openssl.org/)
1. 启用并启动 libvirt 守护进程，例如：

    - 对于基于 systemd 的系统：`systemctl enable libvirtd && systemctl start libvirtd`  
    - 对于基于 init.d 的系统：`/etc/init.d/libvirt-bin start`      


#### &sup1; 取决于您的发行版本，libvirt 访问可能会被默认拒绝，或者可能在每次访问时都需要密码。


您可以使用以下命令对其进行测试：

```shell
virsh -c qemu:///system pool-list
```


如果您收到访问错误信息，请阅读 [https://libvirt.org/acl.html](https://libvirt.org/acl.html) 和 [https://libvirt.org/aclpolkit.html](https://libvirt.org/aclpolkit.html)。


简而言之，如果您的 libvirt 由 Polkit 支持编译（例如：Arch，Fedora 21），您可以按如下方式创建 `/etc/polkit-1/rules.d/50-org.libvirt.unix.manage.rules` 来授予 `$USER` 完全访问 libvirt 的权限。

```shell
sudo /bin/sh -c "cat - > /etc/polkit-1/rules.d/50-org.libvirt.unix.manage.rules" << EOF
```

```conf
polkit.addRule(function(action, subject) {
        if (action.id == "org.libvirt.unix.manage" &&
            subject.user == "$USER") {
                return polkit.Result.YES;
                polkit.log("action=" + action);
                polkit.log("subject=" + subject);
        }
});
EOF
```


如果你的 libvirt 没有使用 Polkit 编译（例如：Ubuntu 14.04.1 LTS），请检查 libvirt unix 套接字上的权限：

```shell
$ ls -l /var/run/libvirt/libvirt-sock
srwxrwx--- 1 root libvirtd 0 févr. 12 16:03 /var/run/libvirt/libvirt-sock

$ usermod -a -G libvirtd $USER

# $USER 需要注销/登录才能被纳入新用户组
```


（请替换 `$USER` 为您的登录名）


#### &sup2; Qemu 将使用一个特定的用户运行。它必须有访问虚拟机驱动的权限。


VM（CoreOS 磁盘镜像，Kubernetes 二进制文件，cloud-init 文件等）所需的所有磁盘驱动器资源都放在 `./cluster/libvirt-coreos/libvirt_storage_pool` 中。


由于我们使用 libvirt 的 `qemu:///system` 实例，qemu 将运行在与您的用户不同的特定 `user:group` 下。这是在 `/etc/libvirt/qemu.conf` 中配置的。该 qemu 用户必须有访问 libvirt 存储池的权限。


如果您的 `$HOME` 目录是全局可读的，那么将一切顺利。但如果您的 `$HOME` 目录是私有的，`cluster/kube-up.sh` 将会失败，并显示如下错误信息：

```shell
error: Cannot access storage file '$HOME/.../kubernetes/cluster/libvirt-coreos/libvirt_storage_pool/kubernetes_master.img' (as uid:99, gid:78): Permission denied
```


为了解决这个问题，您有几种可能的选择：


* 在 `cluster/libvirt-coreos/config-default.sh` 中设置 `POOL_PATH` 为一个这样的目录：
  * 由具有大量可用磁盘空间的文件系统支持
  * 您的用户具有写入权限；
  * qemu 用户可以访问。
* 授予 qemu 用户访问存储池的权限。


在 Arch 上：

```shell
setfacl -m g:kvm:--x ~
```


### 安装


默认情况下，libvirt-coreos 安装程序将创建由一个 Kubernetes master 节点和 3 个 node 节点组成的集群。由于 VM 驱动使用了 Copy-on-Write、memory ballooning 和 KSM，将会有许多的资源超配。


在 CoreOS 上安装 libvirt Kubernetes 集群既可以使用自动方式，也可以使用手动的、自定义的方式。


#### 自动安装

在 [https://get.k8s.io]( https://get.k8s.io) 上有一个自动安装脚本，它将下载 Kubernetes 的压缩包，并在脚本创建的本地 CoreOS 实例上生成一个 Kubernetes 集群。要运行此脚本，请使用 wget 或 curl，并将 KUBERNETES_PROVIDER 环境变量设置为 libvirt-coreos：

```shell
export KUBERNETES_PROVIDER=libvirt-coreos; wget -q -O - https://get.k8s.io | bash
```


这是这个命令的 curl 版本：

```shell
export KUBERNETES_PROVIDER=libvirt-coreos; curl -sS https://get.k8s.io | bash
```


该脚本下载并解压 tar 包，然后在 CoreOS 实例上生成一个 Kubernetes 集群，该集群具有以下特征：

- 一共有 4 个 KVM/QEMU 实例
- 1 个作为 Kubernetes master 节点的实例
- 3 个作为 Kubernetes minion 节点的实例

如果希望使用自定义配置运行集群，请按照手动安装说明进行操作。

#### 手动安装

要启动本地集群，请打开一个 shell 并运行：

```shell
cd kubernetes

export KUBERNETES_PROVIDER=libvirt-coreos
cluster/kube-up.sh
```


`KUBERNETES_PROVIDER` 环境变量告诉所有的集群管理脚本使用哪个变量。如果您忘记设置此项，则假设您正运行在 Google Compute Engine 上。

`NUM_NODES` 环境变量可以被用来指定要启动的节点的数量。如果没有设置，则节点数量默认为 3。


可以设置 `KUBE_PUSH` 环境变量来指定必须在集群上部署哪些 Kubernetes 二进制文件。其可能的值有：

* `release` （如果没有设置 `KUBE_PUSH`，则默认使用该值）将部署 `_output/release-tars/kubernetes-server-….tar.gz` 包中的二进制文件。这是用 `make release` 或 `make release-skip-tests` 构建。
* `local` 将部署 `_output/local/go/bin` 包中的二进制文件。这些文件是用 `make` 构建的。


### 管理

您可以通过运行以下命令检查节点是否已经启动：

```shell
$ virsh -c qemu:///system list
 Id    Name                           State
----------------------------------------------------
 15    kubernetes_master              running
 16    kubernetes_node-01             running
 17    kubernetes_node-02             running
 18    kubernetes_node-03             running
 ```


您可以通过以下命令检查 Kubernetes 集群是否正在运行：

```shell
$ kubectl get nodes
NAME                STATUS     AGE     VERSION
192.168.10.2        Ready      4h      v1.6.0+fff5156
192.168.10.3        Ready      4h      v1.6.0+fff5156
192.168.10.4        Ready      4h      v1.6.0+fff5156
```


VM 正在运行 [CoreOS](https://coreos.com/)。
您的 ssh 密钥已经被推送到了 VM。（将查找 ~/.ssh/id_*.pub）。
用来连接到 VM 的用户是 `core`。
master 节点的 IP 是 192.168.10.1。
node 节点的 IP 是 192.168.10.2 及以上。


连接到 `kubernetes_master`：

```shell
ssh core@192.168.10.1
```


连接到 `kubernetes_node-01`：

```shell
ssh core@192.168.10.2
```

### 使用 `kube-*` 脚本与 Kubernetes 集群交互

以下所有命令都假定您已经正确设置了 `KUBERNETES_PROVIDER`：

```shell
export KUBERNETES_PROVIDER=libvirt-coreos
```


建立一个由 5 个节点组成的 libvirt-CoreOS 集群

```shell
NUM_NODES=5 cluster/kube-up.sh
```


销毁 libvirt-CoreOS 集群

```shell
cluster/kube-down.sh
```


用 `make release` 或 `make release-skip-tests` 生成一个新的 Kubernetes 版本来更新 libvirt-CoreOS 集群：

```shell
cluster/kube-push.sh
```


使用 `make` 生成的本地构建的 Kubernetes 二进制文件更新 libvirt-CoreOS 集群：

```shell
KUBE_PUSH=local cluster/kube-push.sh
```


与集群交互

```shell
kubectl ...
```


### 故障排除


#### !!! Cannot find kubernetes-server-linux-amd64.tar.gz（!!! 找不到 kubernetes-server-linux-amd64.tar.gz 文件）

构建发布压缩包：

```shell
make release
```


#### Can't find virsh in PATH, please fix and retry.（在 PATH 中找不到 virsh，请修复并重试。）

安装 libvirt


在 Arch 上：

```shell
pacman -S qemu libvirt
```


在 Ubuntu 14.04 上：

```shell
aptitude install qemu-system-x86 libvirt-bin
```


在 Fedora 21 上：

```shell
yum install qemu libvirt
```


#### error: Failed to connect socket to '/var/run/libvirt/libvirt-sock': No such file or directory（错误：无法将套接字连接到 '/var/run/libvirt/libvirt-sock'，没有该文件或目录）


启动 libvirt 守护进程


在 Arch 上：

```shell
systemctl start libvirtd virtlogd.socket
```


`virtlogd.socket` 没有随 libvirtd 守护进程启动。如果启用了 `libvirtd.service`，他将被链接并在下次启动时被自动启动。


在 Ubuntu 14.04 上：

```shell
service libvirt-bin start
```


#### error: Failed to connect socket to '/var/run/libvirt/libvirt-sock': Permission denied（错误：无法将套接字连接到 '/var/run/libvirt/libvirt-sock'：权限被拒绝）


修复 libvirt 访问权限（记住要修改 `$USER`）


在 Arch 和 Fedora 21 上：

```shell
cat > /etc/polkit-1/rules.d/50-org.libvirt.unix.manage.rules <<EOF
```

```conf
polkit.addRule(function(action, subject) {
        if (action.id == "org.libvirt.unix.manage" &&
            subject.user == "$USER") {
                return polkit.Result.YES;
                polkit.log("action=" + action);
                polkit.log("subject=" + subject);
        }
});
EOF
```


在 Ubuntu 上：

```shell
usermod -a -G libvirtd $USER
```


#### error: Out of memory initializing network (virsh net-create...)（错误：初始化网络时内存不足（virsh net-create...））


确保安装 ebtables 之后重启了 libvirtd。


## 支持级别


IaaS Provider        | Config. Mgmt | OS     | Networking  | Docs                                              | Conforms | Support Level
-------------------- | ------------ | ------ | ----------  | ---------------------------------------------     | ---------| ----------------------------
libvirt/KVM          | CoreOS       | CoreOS | libvirt/KVM | [docs](/docs/getting-started-guides/libvirt-coreos/)                         |          | Community ([@lhuard1A](https://github.com/lhuard1A))



有关所有解决方案的支持级别信息，请查看 [解决方案表格](/docs/getting-started-guides/#table-of-solutions) 图表。

