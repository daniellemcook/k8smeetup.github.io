---
cn-approvers:
- tianshapjq
title: Decommissioning
---


{% capture overview %}

本页面将向您展示如何合理地销毁一个集群。
{% endcapture %}


{% capture prerequisites %}

本页面假设您有一个使用 Juju 部署的正在运行的集群。


**警告：** 当您到达这一步时，您应该已经把您集群的相关内容进行了备份；这部分是为了彻底销毁一个集群。
{. warning}

{% endcapture %}

{% capture steps %}

## 删除 Juju 模型
建议使用各自的模型来部署独立的 Kubernetes 集群，以便能够清晰地分离每个环境。如果想要删除一个集群，首先需要通过命令 `juju list-models` 找到其对应的模型。控制器为其自身预留了 `admin` 这个模型。如果您没有为您的模型命名，则可能会显示为 `default`。

```
$ juju list-models
Controller: aws-us-east-2

Model       Cloud/Region   Status     Machines  Cores  Access  Last connection
controller  aws/us-east-2  available         1      2  admin   just now
my-kubernetes-cluster*    aws/us-east-2  available        12     22  admin   2 minutes ago
```


然后您可以通过删除模型来销毁模型内的集群：

    juju destroy-model my-kubernetes-cluster
    
```
$ juju destroy-model my-kubernetes-cluster
WARNING! This command will destroy the "my-kubernetes-cluster" model.
This includes all machines, applications, data and other resources.

Continue [y/N]? y
Destroying model
Waiting on model to be removed, 12 machine(s), 10 application(s)...
Waiting on model to be removed, 12 machine(s), 9 application(s)...
Waiting on model to be removed, 12 machine(s), 8 application(s)...
Waiting on model to be removed, 12 machine(s), 7 application(s)...
Waiting on model to be removed, 12 machine(s)...
Waiting on model to be removed...
$
```


这将会彻底破坏并销毁所有节点。您可以通过运行 `juju status` 命令来确认是否所有节点都已销毁。


如果您使用的是公有云环境，则会终止这些实例。如果您使用的是 MAAS 裸机，则会释放这些节点，可以清空磁盘，关闭机器，然后将它们返回到可用的计算机池中进行部署。


## 清理控制器


如果您的控制器没有用于其它途径，您还需要删除控制器实例：

```
$ juju list-controllers
Use --refresh flag with this command to see the latest information.

Controller      Model  User   Access     Cloud/Region   Models  Machines    HA  Version
aws-us-east-2*  -      admin  superuser  aws/us-east-2       2         1  none  2.0.1  

$ juju destroy-controller aws-us-east-2 
WARNING! This command will destroy the "aws-us-east-2" controller.
This includes all machines, applications, data and other resources.

Continue? (y/N):y
Destroying controller
Waiting for hosted model resources to be reclaimed
All hosted models reclaimed, cleaning up controller machines
$ 
```
{% endcapture %}

{% include templates/task.md %}
