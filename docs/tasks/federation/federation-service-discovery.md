---
approvers:
- bprashanth
- quinton-hoole
title: 使用联邦服务进行跨集群服务发现
cn-approvers:
- du2016
---

本文介绍了如何使用 Kubernetes 联邦服务跨多个 Kubernetes 集群部署通用服务。
这使您可以轻松地为 Kubernetes 应用程序实现跨集群服务发现和可用区容错。

* TOC
{:toc}


## 前提要求


本指南假定您有一个已安装运行的 Kubernetes 集群联邦，如果没有，请转到[集群联邦管理指南](/docs/admin/federation/)
以了解如何启动一个集群联邦(或让您的集群管理员安装一个)。其他教程，
例如 Kelsey Hightower 的[这个教程](https://github.com/kelseyhightower/kubernetes-cluster-federation)也可以帮助您。


一般来说，你应对[ Kubernetes 的工作原理](/docs/setup/)有一个基本了解，特别是[服务](/docs/concepts/services-networking/service/)。


## 概览

联邦服务的创建方式和传统[Kubernetes Services](/docs/concepts/services-networking/service/)非常相似，
只需进行一次 API 调用即可指定所需的服务属性。在联邦服务环境下，此API调用将定向到联邦API端点，
而不是 Kubernetes 集群 API 端点，联邦服务的API与传统 Kubernetes Services 的API 100％兼容


一旦创建，联邦服务自动:

1. 在集群联邦底层的每个集群中创建对应的 Kubernetes 服务，
2. 监视那些服务 “shards” (以及它们所在的集群)的健康状况，
3. 在公共 DNS 提供商(例如 Google Cloud DNS 或 AWS Route 53 )中管理一组 DNS 记录，
从而确保您的联邦服务的客户端可以随时无缝地找到合适的健康服务端点,即使在集群可用区或区域中断的情况下也是如此。


在集群联邦内部的客户端(即 Pod )，如果集群联邦存在且健康，会自动在所在集群中找到集群联邦的本地分片，
如果不存在，将会寻找最接近健康的分片。


## 混合云功能

Kubernetes 集群联邦可以包含运行在不同云提供商(例如 Google Cloud ，AWS )中的集群,以及本地私有云(例如在 OpenStack 上)。
只需在适当的云提供商 和/或 地区创建您需要的所有集群，并且向联邦 API Server 注册每个集群的API端点和凭证(参考[集群联邦管理指南](/docs/admin/federation/)了解详细信息)。


之后，您的应用程序和服务可以跨越不同的集群和云提供商，详情如下所述。


## 创建联邦服务



通常这样完成，例如:

``` shell
kubectl --context=federation-cluster create -f services/nginx.yaml
```


'--context=federation-cluster'参数告诉 kubectl 使用合适的凭证将请求提交给联邦 API 端点。
如果你还没有配置这样的上下文，访问[集群联邦管理指南](/docs/admin/federation/)或[管理教程](https://github.com/kelseyhightower/kubernetes-cluster-federation)
其中之一了解如何做到这一点。


如上所述，联邦服务将自动在联邦底层的所有集群中创建和维护对应的 Kubernetes 服务。


您可以通过检查每个基础集群来验证这一点，例如:

``` shell
kubectl --context=gce-asia-east1a get services nginx
NAME      CLUSTER-IP     EXTERNAL-IP      PORT(S)   AGE
nginx     10.63.250.98   104.199.136.89   80/TCP    9m
```


上面假定您的客户端中为您的集群在该区域中配置了一个名为'gce-asia-east1a'的上下文,
底层服务的名称和命名空间将自动匹配上面创建的联邦服务的名称和命名空间(
如果碰巧在这些集群中有已经存在的相同名称和命名空间的服务，
它们将被联邦自动采用并更新以符合您联邦服务的规范 - 无论哪种方式，最终结果都将一样)。



联邦服务的状态将自动反映底层 Kubernetes 服务的实时状态，例如:

``` shell
$kubectl --context=federation-cluster describe services nginx

Name:                   nginx
Namespace:              default
Labels:                 run=nginx
Selector:               run=nginx
Type:                   LoadBalancer
IP:                     10.63.250.98
LoadBalancer Ingress:   104.197.246.190, 130.211.57.243, 104.196.14.231, 104.199.136.89, ...
Port:                   http    80/TCP
Endpoints:              <none>
Session Affinity:       None
No events.
```


请注意，联邦服务的'LoadBalancer Ingress'地址与所有基础 Kubernetes 服务的'LoadBalancer Ingress'地址相对应(
一旦这些被分配 - 这可能需要几秒钟)。为了集群间和提供商间服务分片的网络正常工作,
您的服务需要有一个外部可见的IP地址。[Service Type:Loadbalancer](/docs/concepts/services-networking/service/#type-loadbalancer)
通常用于此, 尽管也有其他选项(例如 [External IP's](/docs/concepts/services-networking/service/#external-ips))存在。


还要注意，我们还没有配置任何后端 Pod 来接收指向这些地址的网络流量(即'Service Endpoints')，
所以联邦服务还没有认为这些服务是健康的服务分片，因此还没有将这些服务添加到这个联邦服务的 DNS 记录中(后面更多关于这方面的内容)。


## 添加后端 pod


为了使底层服务分片健康，我们需要在服务后面添加后端 Pod 。
这目前是直接针对底层集群的API端点完成的(尽管将来联邦服务器将能够使用单个命令为您完成所有这些工作，以节省您的麻烦)。
例如，要在13个基础集群中创建后端 Pod ：

``` shell
for CLUSTER in asia-east1-c asia-east1-a asia-east1-b \
                        europe-west1-d europe-west1-c europe-west1-b \
                        us-central1-f us-central1-a us-central1-b us-central1-c \
                        us-east1-d us-east1-c us-east1-b
do
  kubectl --context=$CLUSTER run nginx --image=nginx:1.11.1-alpine --port=80
done
```


请注意，`kubectl run`会自动添加`run = nginx`标签，从而将后端Pod与对应的service相关联。


## 验证公共 DNS 记录

一旦上面的 Pod 成功启动，并开始监听连接，Kubernetes 将报告 Pod 作为在该集群服务的健康端点(通过自动健康检查)。
集群联邦将依次考虑这些服务“分片”中的每一个都是健康的，并通过自动配置相应的公共 DNS 记录以将其放置在服务中。
您可以使用您的首选接口到您配置的 DNS 提供商来验证这一点。例如，如果您的联邦配置为使用Google Cloud DNS，
并且托管 DNS 域名为'example.com':

``` shell
$ gcloud dns managed-zones describe example-dot-com
creationTime: '2016-06-26T18:18:39.229Z'
description: Example domain for Kubernetes Cluster Federation
dnsName: example.com.
id: '3229332181334243121'
kind: dns#managedZone
name: example-dot-com
nameServers:
- ns-cloud-a1.googledomains.com.
- ns-cloud-a2.googledomains.com.
- ns-cloud-a3.googledomains.com.
- ns-cloud-a4.googledomains.com.
```

``` shell
$ gcloud dns record-sets list --zone example-dot-com
NAME                                                            TYPE      TTL     DATA
example.com.                                                    NS        21600   ns-cloud-e1.googledomains.com., ns-cloud-e2.googledomains.com.
example.com.                                                    OA        21600   ns-cloud-e1.googledomains.com. cloud-dns-hostmaster.google.com. 1 21600 3600 1209600 300
nginx.mynamespace.myfederation.svc.example.com.                 A         180     104.197.246.190, 130.211.57.243, 104.196.14.231, 104.199.136.89,...
nginx.mynamespace.myfederation.svc.us-central1-a.example.com.   A         180     104.197.247.191
nginx.mynamespace.myfederation.svc.us-central1-b.example.com.   A         180     104.197.244.180
nginx.mynamespace.myfederation.svc.us-central1-c.example.com.   A         180     104.197.245.170
nginx.mynamespace.myfederation.svc.us-central1-f.example.com.   CNAME     180     nginx.mynamespace.myfederation.svc.us-central1.example.com.
nginx.mynamespace.myfederation.svc.us-central1.example.com.     A         180     104.197.247.191, 104.197.244.180, 104.197.245.170
nginx.mynamespace.myfederation.svc.asia-east1-a.example.com.    A         180     130.211.57.243
nginx.mynamespace.myfederation.svc.asia-east1-b.example.com.    CNAME     180     nginx.mynamespace.myfederation.svc.asia-east1.example.com.
nginx.mynamespace.myfederation.svc.asia-east1-c.example.com.    A         180     130.211.56.221
nginx.mynamespace.myfederation.svc.asia-east1.example.com.      A         180     130.211.57.243, 130.211.56.221
nginx.mynamespace.myfederation.svc.europe-west1.example.com.    CNAME     180     nginx.mynamespace.myfederation.svc.example.com.
nginx.mynamespace.myfederation.svc.europe-west1-d.example.com.  CNAME     180     nginx.mynamespace.myfederation.svc.europe-west1.example.com.
... etc.
```


注意：如果您的联邦配置为使用 AWS Route53，则可以使用其中一个等效的AWS工具，例如：

``` shell
$ aws route53 list-hosted-zones
```
或
``` shell
$ aws route53 list-resource-record-sets --hosted-zone-id Z3ECL0L9QLOVBX
```


无论您使用哪种 DNS 服务程序,任何 DNS 查询工具(例如'dig'或'nslookup')允许您查看联邦为您创建的记录。
请注意，您应该直接在您的 DNS 提供商处指出这些工具(例如`dig @ns-cloud-e1.googledomains.com ...`),
或者根据您配置的 TTL 预计延迟(默认为180秒)看到更新,由于中间 DNS 服务器的缓存.


### 关于上面的例子的一些说明

1. 请注意，每个至少有一个健康后端端点的服务分片有一个正常('A')记录。
例如，在 us-central1-a 中，104.197.247.191是该区域中服务分片的外部IP地址，
而在 asia-east1-a 中，地址是130.211.56.221。
2. 同样，还有区域'A'记录，包括该地区所有健康的分片。
例如，'us-central1'。这些区域记录对于不具有特定区域首选项的客户端以及下述局部自动化和故障转移机制的构建块非常有用。
3. 对于当前没有健康后端端点的区域，使用CNAME('Canonical Name')记录将这些查询别名(自动重定向)到下一个最接近的健康区域。
在本例中，us-central1-f 中的服务分片当前没有健康的后端端点(即 Pod )，
因此已创建CNAME记录自动将查询重定向到该区域中的其他分片(本例中为 us-central1 )。
4. 同样，如果封闭区域内没有健康的碎片，
搜索就会进一步扩展。在 europe-west1-d 可用性区域，没有健康的后端，
所以查询被重定向到更广泛的 europe-west1 (其也没有健康的后端Pod)，
并且继续到全局健康的地址('nginx.mynamespace .myfederation.svc.example.com。')。


上述 DNS 记录集自动保持与联邦服务系统全局所有服务分片的当前健康状态同步。
 DNS 解析器库(由所有客户端调用)自动遍历 'CNAME' 和 'A' 记录的层次结构，
以返回正确的健康 IP 地址集。然后，
客户端可以选择任何一个返回的地址来启动网络连接(如果需要，可以自动切换到其他等效地址之一)。


## 发现联邦服务


### 从联邦集群内部的 pod


默认情况下，Kubernetes 集群预先配置了集群本地DNS服务器('KubeDNS'),以及智能构建的DNS搜索路径，
这些路径一起确保您在 Pod 内运行的软件发出的“myservice”，“myservice.mynamespace”，
“bobsservice.othernamespace”等DNS查询会自动扩展并正确解析到在本地集群中运行的服务相应的服务IP。



随着联邦服务和跨集群服务发现的引入，此概念将扩展到涵盖在全局范围内跨集群联邦的任何其他集群中运行的 Kubernetes 服务。
要充分利用此扩展范围，请使用稍微不同的 DNS 名称(形式"<服务名称>.<命名空间>.<联邦名称>"，
例如 myservice.mynamespace.myfederation )来解析联邦服务。
你没有明确地选择这种行为时使用不同的 DNS 名称还可避免现有应用程序意外地穿越跨区域或跨区域网络，
并且您因此可能会收到不必要的网络费用或延迟。


因此，使用我们的NGINX演示上面的服务，以及刚刚描述的联邦服务DNS名称表单，我们来看一个例子：
`us-central1-f`可用区中的集群中的Pod需要连接我们的 NGINX 服务。现在可以使用服务的联邦 DNS 名称
```"nginx.mynamespace.myfederation"```
而不是使用服务在传统集群本地的 DNS (```"nginx.mynamespace"```, 将自动扩展到
```"nginx.mynamespace.svc.cluster.local"```)名称，
这将自动扩展，并解析到我的 NGINX 服务的最接近健康的分片，无论在世界任何地方。
如果在本地集群中存在健康的分片，该服务的集群本地(通常 10.x.y.z)IP地址将被返回(由集群本地 KubeDNS )，
这几乎完全等同于非联邦服务解决方案(接近因为 KubeDNS 实际上为本地联邦服务返回一个 CNAME 和一个 A 记录，
但是应用程序会忽略这个小的技术差异)。


但是如果这个服务在本地集群中不存在的话(或者服务存在,但是没有健康的后端 pod )，那么 DNS 查询会自动扩展为
```"nginx.mynamespace.myfederation.svc.us-central1-f.example.com”```
(即逻辑上"找到离我的可用区域最近的其中一个分片的外部 IP "),这个扩展是由 KubeDNS 自动执行的，它返回相关的 CNAME 记录。
在上面的例子中，这导致了自动遍历 DNS 记录的层次结构，并且在本地 us-central1 区域的联邦服务的外部IP之一处结束
(即 104.197.247.191,104.197.244.180 or 104.197.245.170)。 


当然，可以通过明确指定适当的DNS名称，而不是依靠DNS的自动扩展，
明确地定位可用区域和Pod以外的区域中的服务分片，而不是依靠自动 DNS 扩展。例如，
"nginx.mynamespace.myfederation.svc.europe-west1.example.com" 将解析欧洲所有目前健康的服务分片，
即使发布查询的 Pod 位于美国，也不管在美国是否有健康的服务碎片。这对于远程监控和其他类似的应用是有用的。


### 来自集群联邦之外的其他客户端


上述讨论大部分同样适用于外部客户端，只是所述的自动 DNS 扩展已不再可用。
因此，外部客户端需要指定联邦服务的完全限定的 DNS 名称之一，是区域名称，
地区名称或全球名称。出于方便的原因，在服务中手动配置其他静态 CNAME 记录通常是一个好主意，例如：

``` shell
eu.nginx.acme.com        CNAME nginx.mynamespace.myfederation.svc.europe-west1.example.com.
us.nginx.acme.com        CNAME nginx.mynamespace.myfederation.svc.us-central1.example.com.
nginx.acme.com           CNAME nginx.mynamespace.myfederation.svc.example.com.
```


这样，您的客户就可以使用左侧的简短表格，并且可以自动将其路由到本国最近的健康分片。
Kubernetes 集群联邦自动处理所有必需的故障转移。未来的版本将进一步改善。


## 处理后端 pod 和整个集群的失败


标准 Kubernetes 服务 cluster-IP 已经确定无响应的单独Pod端点自动以低延迟(几秒钟)退出服务。
此外，如上所述，Kubernetes 集群联邦系统会自动监控集群的健康状况，
以及您的联邦服务的所有分片背后的端点，根据需要采取分片进入和退出服务(例如当服务后面的所有端点，
或者整个集群或可用区域都停止运行时，或者相反，从停机状态恢复)。由于 DNS 缓存固有的延迟(
默认情况下，联邦服务 DNS 记录的缓存超时或 TTL 配置为3分钟，但可以进行调整)。
在灾难性失败的情况下，所有客户可能需要花费很长时间才能完全故障切换到另一个集群。
但是，考虑到每个区域服务端点可以返回的离散IP地址的数量(例如上面 us-central1，有三个选择)
许多客户端将自动故障切换到其中一个替代IP的时间少于给定的适当配置。


## 故障排查

#### 我无法连接到我的集群联邦 API
检查你的:

1. 客户端(通常为 kubectl )配置正确(包括 API 端点和登录证书)。
2. 集群联邦 API 服务器正在运行并且网络可达。


请参阅[集群联邦管理指南](/docs/admin/federation/)了解如何正确启动集群联邦身份验证
(或让集群管理员为您执行此操作),以及如何正确配置客户端


#### 我可以通过集群联邦 API 成功创建一个联邦服务，但在我的底层集群中没有创建对应的服务
检查：

1. 您的集群在集群联邦 API (`kubectl describe clusters`)中正确注册。
2. 你的集群都是 'Active' 的。这意味着集群联邦系统能够连接和验证集群端点。如果不是，请查阅 federation-controller-manager pod 的日志以确定可能发生的故障。
```kubectl --namespace=federation logs $(kubectl get pods --namespace=federation -l module=federation-controller-manager -o name)```
3. 该集群提供给集群联邦 API 的登录凭证具有正确的授权和配额，以在集群中的相关命名空间中创建服务。如果不是这种情况，您应该再次看到相关的错误消息提供了上述日志文件中的更多细节。
4. 是否有其他错误阻止了服务创建操作的成功(在`kubectl logs federation-controller-manager --namespace federation'的输出中查找'service-controller`错误)。


#### 我可以成功创建联邦服务，但是在我的 DNS 提供程序中没有创建匹配的DNS记录
检查:

1.您的联邦名称，DNS 提供程序，DNS 域名配置正确。请参阅[联邦集群管理指南](/docs/admin/federation/)或[教程](https://github.com/kelseyhightower/kubernetes-cluster-federation)以了解
如何配置集群联邦身份验证系统的 DNS 提供程序(或让您的集群管理员为您执行此操作)。
2.确认集群联邦的服务控制器已经成功连接到所选的 DNS 提供程序并进行身份验证(在`kubectl logs federation-controller-manager --namespace federation`的输出中查找`service-controller` errors 或 successes)。


#### 匹配到我在 DNS 提供商中创建的 DNS 记录，但客户端无法解析这些记录
检查:

1. 管理您的联邦 DNS 域的 DNS 注册管理器已正确配置到指向您配置的DNS提供商的域名服务器。
例如，请参阅 [Google Domains文档](https://support.google.com/domains/answer/3290309?hl=zh_CN&ref_topic=3251230) 和 [Google云端DNS文档](https://cloud.google.com/dns/update-name-servers) ，或从您的域名注册商或 DNS 提供商获得同样的指导。

#### 此故障排除指南并没有帮助我解决我的问题

1. 请使用我们的[支持渠道](/docs/tasks/debug-application-cluster/troubleshooting/)寻求帮助。


## 了解更多信息

* [Federation proposal](https://git.k8s.io/community/contributors/design-proposals/multicluster/federation.md)详细介绍了激发集群联邦工作的用例.
