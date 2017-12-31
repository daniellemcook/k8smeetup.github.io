---
approvers:
- bprashanth
title: VMware Photon Controller
---


下面的例子将使用VMware的Photon Controller创建一个kubernetes集群。
该集群将拥有一个kubernetes master和三个kubernetes节点。

* TOC
{:toc}



### 前提条件

1. 你需要拥有[VMware Photon Controller](http://vmware.github.io/photon-controller/) 部署的管理员访问权限（Administrator access）。
（管理员访问权限只在初始化安装的时候需要，集群的实际创建可以由任何人完成.）

2. 你需要在准备运行kube-up的机器上安装[Photon Controller CLI](https://github.com/vmware/photon-controller-cli)。
如果你已经安装了go，这个就很容易安装：

		go get github.com/vmware/photon-controller-cli/photon

3. 需要安装mkisofs 。这个安装过程会创建一个CD-ROM ISO镜像，并使用cloud-init来引导VM。
如果你使用的是Mac，你可以使用brew来安装(http://brew.sh/):：

		brew install cdrtools

4. 还需要安装几个常用工具：`ssh`, `scp`, `openssl`.

5. 需要安装ssh 公钥。这个公钥允许你访问VM的用户账号，"kube"。

6. 直接获取或构建一个二进制发行版(/docs/getting-started-guides/binary_release)。


### 下载VM镜像

下载一个预编译的Debian 8.2 VMDK，这将是我们使用的基础镜像：

```shell
curl --remote-name-all https://s3.amazonaws.com/photon-platform/artifacts/OS/debian/debian-8.2.vmdk
```


这个基础Debian 8.2镜像还安装了：

* openssh-server
* open-vm-tools
* cloud-init


### 配置Photon Controller:


为了部署kubernetes，你需要在Photon Controller上配置：
* 一个拥有相关资源标签（ticket）的租户
* 一个位于该租户内的项目
* VM和描述VM特点的磁盘风格
* 一个镜像：我们将使用上面那个镜像

在完成以上配置之后，你还需要使用租户，项目，风格和镜像的名字来配置cluster/photon-controller/config-common.sh文件。

你如果喜欢，也可以使用已提供的cluster/photon-controller/setup-prereq.sh脚本来创建这些配置。
假设你的Photon controller的IP地址是192.0.2.2（视情况可以修改），已下载的镜像是kebe.vmdk，你可以运行：

```shell
photon target set https://192.0.2.2
photon target login ...credentials...
cluster/photon-controller/setup-prereq.sh https://192.0.2.2 kube.vmdk
```

setup-prereq.sh脚本会基于kube-up使用的同一个配置文件cluster/photon-controller/config-common.sh来
创建租户，项目，风格和镜像。需要注意的是，这个脚本会创建一个资源标签来限制每个租户可创建VM的数量。
你可以根据实际Photon controller的部署，修改config-comment.sh中资源标签的配置。



### 配置kube-up

配置kube-up与photon controller的交互涉及到两个文件：

1.	cluster/photon-controller/config-common.sh包含了最常用的参数，包括租户、相互和镜像的名字。

2.	cluster/photon-controller/config-default.sh包含更多高级参数，比如使用的IP子网，
需要创建节点的数量，要配置哪个kubernetes的组件。

以上两个文件都有相应的文档解释不同的参数。



### 创建你的kubernetes集群

我们通过运行标准的kube-up命令就可以创建自己的kubernetes集群。
如上所述，控制kube-up与photon controller之间交互的参数都是在文件中指定的，而不是在命令行中。

集群部署的时间会因为你创建节点的数量、photon Controller主机和网络的配置而改变。
对一个10个节点的集群而言，时长大概是10到30分钟。

```shell
KUBERNETES_PROVIDER=photon-controller cluster/kube-up.sh
```

一旦你成功到达这一步，你的kubernetes集群与其他集群一样正常工作了。

注意：kube-up在~/.kube/config中为你创建了一个kubernetes配置文件。
这个文件允许你使用kubectl命令。它包含kubernetes master的IP地址和admin用户的密码。
如果你想使用kubernetes基于web的用户接口，你需要这个密码。在这个配置文件中，
你会发现一个类似如下的章节：你的密码就是在这里使用。（注意：本输出是被截取过得：证书数据非常长）

```yaml
- name: photon-kubernetes
  user:
    client-certificate-data: Q2Vyd...
    client-key-data: LS0tL...
    password: PASSWORD-HERE
    username: admin
```


### 移除你的kubernetes集群

移除kubernetes集群的推荐方法就是使用kube-down命令：

```shell
KUBERNETES_PROVIDER=photon-controller cluster/kube-down.sh
```

你的kubernetes集群就是一个VM的集合：如果需要，你可以手动移除他们。



### 让外部可访问服务

在kubernetes中，让外部可访问服务的方法有很多种。
目前，photon-controller支持还不包含对LoadBalancer选项的内嵌式支持。


#### 选择1：  Nodeport

选择之一就是使用在手动部署balancer时使用Nodeport选项。确切来说：

使用Nodeport选项来配置你的服务。例如，本服务使用Nodeport选项。
所有的kubernetes节点将监听某个端口，并将网络流量转发给该服务中的任何pod。
在这种情况下，kubernetes会选择一个随机端口，但是在所有节点上都会选择同一个端口。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-demo-service
  labels:
    app: nginx-demo
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    name: http
  selector:
    app: nginx-demo
```


接下来，创建一个新的单独的VM（或为了高可用性，可使用多个VM）来扮演load balancer。
例如，如果你使用haproxy，你可以使用与底下相似的配置。需要注意的是，
这个例子假设有三个kubernetes节点：你可以根据实际拥有的节点数来调整配置。
还需要注意一点，需要将30144端口替换成kubernetes分配给Nodeport的那个数。

```yaml
frontend nginx-demo
    bind *:30144
    mode http
    default_backend nodes
backend nodes
    mode http
    balance roundrobin
    option forwardfor
    http-request set-header X-Forwarded-Port %[dst_port]
    http-request add-header X-Forwarded-Proto https if { ssl_fc }
    option httpchk HEAD / HTTP/1.1\r\nHost:localhost
    server web0 192.0.2.2:30144 check
    server web1 192.0.2.3:30144 check
    server web2 192.0.2.4:30144 check
```


选择2： ingress controller

使用ingress controller也是一个合适的解决方法。需要注意的是，在生产环境中，
它也需要一个额外的load balancer。但是，它管理起来会更容易，因为它不需要你像之前一样手动更新load balancer配置。



### 细节

#### 登录VM

在VM被创建的时候，也会创建一个kube用户（使用cloud-init）。Kube用户的密码与
你kubernetes master的管理员密码是一样的，可以在你的kubernetes配置文件中找到：
上面已经描述过如何找。Kube用户也会授权你的ssh公钥登录。这个公钥在安装过程中使用，用来避免密码输入。

VM的确还有root用户，但ssh到root用户被禁用。


### 网络

Kubernetes集群使用kube-proxy来配置有iptables的overlay网络。
目前我们不支持其他overlay网络，比如Weave或Calico。


## 支持级别


IaaS Provider        | Config. Mgmt | OS     | Networking  | Docs                                              | Conforms | Support Level
-------------------- | ------------ | ------ | ----------  | ---------------------------------------------     | ---------| ----------------------------
Vmware Photon        | Saltstack    | Debian | OVS         | [docs](/docs/getting-started-guides/photon-controller)                      |          | Community ([@alainroy](https://github.com/alainroy))