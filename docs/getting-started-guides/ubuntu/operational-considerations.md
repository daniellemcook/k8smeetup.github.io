---
title: 操作注意事项
---


{% capture overview %}

本文为管理长期运行的集群的人员提供一些建议和提示
{% endcapture %}
{% capture prerequisites %}

本文假定您对 Juju 和 Kubernetes 有了基本的理解。
{% endcapture %}

{% capture steps %}


## 管理 Juju

### 改变您的控制节点数量


Juju 控制器：

* 需要大概 2 到 2.5GB 的 RAM 来运行。
* 使用 MongoDB 数据库作为集群配置和状态的存储后端。这个数据库可能增长很快，也可能是实例中 CPU 周期的最大消费者
* 汇总和存储所有服务和单位的日志数据。因此，长期运行的模型需要大量的存储。如果您的目的是保持集群运行，请确保为日志配置至少 64GB。

```
juju bootstrap --contraints "mem=8GB cpu-cores=4 root-disk=128G"
```


Juju 将会选择与您目标云上的约束匹配的最便宜的实例类型。您还可以将 ```实例类型``` 约束与 ```根磁盘``` 结合使用以进行严格控制。对于可用的约束信息，请参阅 [官方文档](https://jujucharms.com/docs/stable/reference-constraints)


关于日志记录的更多信息，请参阅 [日志章节](/docs/getting-started-guides/ubuntu/logging)


### 通过 SSH 连接到控制节点

默认情况下，Juju 将创建一对 SSH 密钥，用于不同单元的自动连接。这些密钥存储在客户端节点的 ```~/.local/share/juju/ssh/```

部署之后，Juju Controller 是一个 "无声单元"，其充当客户端和已部署应用程序之间的代理。尽管如此，SSH 还是有用的。

首先你需要了解你的环境，特别是如果你运行了几个 Juju 模型和控制器。运行

```
juju list-models --all
$ juju models --all
Controller: k8s

Model             Cloud/Region   Status     Machines  Cores  Access  Last connection
admin/controller  lxd/localhost  available         1      -  admin   just now
admin/default     lxd/localhost  available         0      -  admin   2017-01-23
admin/whale*      lxd/localhost  available         6      -  admin   3 minutes ago
```


第一行的 ```Controller: k8s``` 表示您的引导类型。

接着您能看到下面列出 2 个或更多的类型。


* admin/controller 是承载 juju 的所有控制器单元的默认模型
* admin/default 默认情况下被创建为承载用户应用程序的主要模型，例如 Kubernetes 集群
* admin/whale 是一个额外的模型，如果你使用 conjure-up 覆盖在 Juju 之上

现在通过 ssh 进入一个控制节点，首先您要求 Juju 切换上下文，然后就能像普通单元一样使用 ssh：

```
juju switch controller
```


在这个阶段，您也可以查询控制器模型：

```
juju status
Model       Controller  Cloud/Region   Version
controller  k8s           lxd/localhost  2.0.2

App  Version  Status  Scale  Charm  Store  Rev  OS  Notes

Unit  Workload  Agent  Machine  Public address  Ports  Message

Machine  State    DNS           Inst id        Series  AZ
0        started  10.191.22.15  juju-2a5ed8-0  xenial  
```


请注意，如果您是在 HA 模式下进行的引导，则会看到列出了几台机器。

现在 ssh 进入控制器遵循与经典 Juju 命令相同的语义：

```
$ juju ssh 0
Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.8.0-34-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.


Last login: Tue Jan 24 16:38:13 2017 from 10.191.22.1
ubuntu@juju-2a5ed8-0:~$ 
```


如果您完成了操作然后向返回到最初的模型，那么退出控制器并且

然后，如果您想要切换回您的集群并且 ssh 进入其他单元，运行

```
juju switch default
```


## 管理您的 Kubernetes 集群

### 运行特权容器


默认情况下，Juju 部署的集群不支持运行特权容器。
如果您需要它们，您必须在 kubernetes-master 和 kubernetes-worker 上启用 ```allow-privileged``` 配置：

```
juju config kubernetes-master allow-privileged=true
juju config kubernetes-worker allow-privileged=true
```


### 私有仓库

通过仓库提供的功能，您可以很容易地创建一个使用 TLS 身份验证的私有 docker 仓库。但是请注意，通过这些功能部署的仓库不是 HA 的；它使用的存储绑定到运行 pod 的 kubernetes 节点上。因此，如果仓库所在的 pod 迁移到另一个节点上，那么你需要重新发布镜像。


#### 使用示例

创建相关的身份验证文件。例如您想要用户 ```userA``` 通过密码 ```passwordA``` 进行身份验证，那么您需要：

```
echo "userA:passwordA" > htpasswd-plain
htpasswd -c -b -B htpasswd userA passwordA
```


（`htpasswd` 程序通过 ```apache2-utils``` 包获得）


假设您的仓库可以通过 ```myregistry.company.com``` 获得，您已经在 ```registry.key``` 文件中拥有了您的 TLS 密钥，并且您的 TLS 身份验证（以 ```myregistry.company.com``` 作为通用名）在 ```registry.crt``` 文件中，那么您可以运行：

```
juju run-action kubernetes-worker/0 registry domain=myregistry.company.com htpasswd="$(base64 -w0 htpasswd)" htpasswd-plain="$(base64 -w0 htpasswd-plain)" tlscert="$(base64 -w0 registry.crt)" tlskey="$(base64 -w0 registry.key)" ingress=true
```


如果您想要删除仓库，只需要运行：

```
juju run-action kubernetes-worker/0 registry delete=true ingress=true
```


{% endcapture %}

{% include templates/task.md %}
