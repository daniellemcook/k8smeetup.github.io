---
cn-approvers:
- linyouchong
title: 使用工作队列进行精细的并行处理
---


* TOC
{:toc}


# 示例：每个 Pod 处理工作队列中多个工作项的 Job


在这个例子中，我们会运行一个存在多个并行工作进程的 Kubernetes Job。您可能希望先了解一些基础的、非并行使用 [Job](/docs/concepts/jobs/run-to-completion-finite-workloads/) 的知识。


在这个例子中，当每个pod被创建时，它会从一个任务队列中获取一个工作单元，处理它，然后重复，直到到达队列的尾部。



下面是这个示例的步骤概述


1. **启动存储服务用于保存工作队列。** 在这个例子中，我们使用 Redis 来存储工作项。在上一个例子中，我们使用了 RabbitMQ。在这个例子中，由于 AMQP 不能为客户端提供一个良好的方法来检测一个有限长度的工作队列是否为空，我们使用了 Redis 和一个自定义的工作队列客户端库。在实践中，您可能会设置一个类似于 Redis 的存储库，并将其同时用于多项任务或其他事务的工作队列。

1. **创建一个队列，然后向其中填充消息。** 每个消息表示一个将要被处理的工作任务。在这个例子中，消息只是一个我们将用于进行长度计算的整数。

1. **启动一个 Job 对队列中的任务进行处理**。这个 Job 启动了若干个 Pod 。每个 Pod 从消息队列中取出一个工作任务，处理它，然后重复，直到到达队列的尾部。



## 启动 Redis


对于这个例子，为了简单起见，我们将启动一个单实例的 Redis。
了解如何部署一个可伸缩、高可用的 Redis 例子，请查看 [Redis 样例](https://github.com/kubernetes/examples/tree/master/guestbook) 


启动一个临时的 Pod 用于运行 Redis 和 一个临时的 service 以便我们能够找到这个 Pod

```shell
$ kubectl create -f docs/tasks/job/fine-parallel-processing-work-queue/redis-pod.yaml
pod "redis-master" created
$ kubectl create -f docs/tasks/job/fine-parallel-processing-work-queue/redis-service.yaml
service "redis" created
```


如果您没有使用本文档库的源代码目录，您可以直接下载 [`redis-pod.yaml`](redis-pod.yaml?raw=true) 和 [`redis-service.yaml`](redis-service.yaml?raw=true) 。


## 使用任务填充队列


现在，让我们往队列里添加一些“任务”。在这个例子中，我们的任务只是一些将被打印出来的字符串。


启动一个临时的可交互的 pod 用于运行 Redis 命令行界面。

```shell
$ kubectl run -i --tty temp --image redis --command "/bin/sh"
Waiting for pod default/redis2-c7h78 to be running, status is Pending, pod ready: false
Hit enter for command prompt
```


现在按回车键，启动 redis 命令行界面，然后创建一个存在若干个工作项的列表。

```
# redis-cli -h redis
redis:6379> rpush job2 "apple"
(integer) 1
redis:6379> rpush job2 "banana"
(integer) 2
redis:6379> rpush job2 "cherry"
(integer) 3
redis:6379> rpush job2 "date"
(integer) 4
redis:6379> rpush job2 "fig"
(integer) 5
redis:6379> rpush job2 "grape"
(integer) 6
redis:6379> rpush job2 "lemon"
(integer) 7
redis:6379> rpush job2 "melon"
(integer) 8
redis:6379> rpush job2 "orange"
(integer) 9
redis:6379> lrange job2 0 -1
1) "apple"
2) "banana"
3) "cherry"
4) "date"
5) "fig"
6) "grape"
7) "lemon"
8) "melon"
9) "orange"
```


因此，这个键为 `job2` 的列表就是我们的工作队列。


注意：如果您还没有正确地配置 Kube DNS，您可能需要将上面的第一步改为 `redis-cli -h $REDIS_SERVICE_HOST`。



创建镜像


现在我们已经准备好创建一个我们要运行的镜像


我们会使用一个带有 redis 客户端的 python 工作程序从消息队列中读出消息。


这里提供了一个简单的 Redis 工作队列客户端库，叫 rediswq.py ([下载](rediswq.py?raw=true))。


Job 中每个 Pod 内的 “工作程序” 使用工作队列客户端库获取工作。如下：

{% include code.html language="python" file="worker.py" ghlink="/docs/tasks/job/fine-parallel-processing-work-queue/worker.py" %}


如果您在使用本文档库的源代码目录，请将当前目录切换到 `docs/tasks/job/fine-parallel-processing-work-queue/`。否则，请点击链接下载 [`worker.py`](worker.py?raw=true)、 [`rediswq.py`](rediswq.py?raw=true) 和 [`Dockerfile`](Dockerfile?raw=true)。然后构建镜像：

```shell
docker build -t job-wq-2 .
```


### Push 镜像


对于 [Docker Hub](https://hub.docker.com/)，请先用您的用户名给镜像打上标签，然后使用下面的命令 push 您的镜像到仓库。请将 `<username>` 替换为您自己的用户名。

```shell
docker tag job-wq-2 <username>/job-wq-2
docker push <username>/job-wq-2
```


您需要将镜像 push 到一个公共仓库或者 [配置集群访问您的私有仓库](/docs/concepts/containers/images/)。


如果您使用的是 [Google Container
Registry](https://cloud.google.com/tools/container-registry/)，请先用您的 project ID 给您的镜像打上标签，然后 push 到 GCR 。请将 `<project>` 替换为您自己的 project ID

```shell
docker tag job-wq-2 gcr.io/<project>/job-wq-2
gcloud docker -- push gcr.io/<project>/job-wq-2
```


## 定义一个 Job


这是 job 定义：

{% include code.html language="yaml" file="job.yaml" ghlink="/docs/tasks/job/fine-parallel-processing-work-queue/job.yaml" %}


请确保将 job 模板中的 `gcr.io/myproject` 更改为您自己的路径。


在这个例子中，每个 pod 处理了队列中的多个项目，直到队列中没有项目时便退出。因为是由工作程序自行检测工作队列是否为空，并且 Job 控制器不知道工作队列的存在，所以依赖于工作程序在完成工作时发出信号。工作程序以成功退出的形式发出信号表示工作队列已经为空。所以，只要有任意一个工作程序成功退出，控制器就知道工作已经完成了，所有的 Pod 将很快会退出。因此，我们将 Job 的 completion count 设置为 1 。尽管如此，Job 控制器还是会等待其它 Pod 完成。



## 运行 Job


现在运行这个 Job ：

```shell
kubectl create -f ./job.yaml
```


稍等片刻，然后检查这个 Job。

```shell
$ kubectl describe jobs/job-wq-2
Name:             job-wq-2
Namespace:        default
Selector:         controller-uid=b1c7e4e3-92e1-11e7-b85e-fa163ee3c11f
Labels:           controller-uid=b1c7e4e3-92e1-11e7-b85e-fa163ee3c11f
                  job-name=job-wq-2
Annotations:      <none>
Parallelism:      2
Completions:      <unset>
Start Time:       Mon, 11 Jan 2016 17:07:59 -0800
Pods Statuses:    1 Running / 0 Succeeded / 0 Failed
Pod Template:
  Labels:       controller-uid=b1c7e4e3-92e1-11e7-b85e-fa163ee3c11f
                job-name=job-wq-2
  Containers:
   c:
    Image:              gcr.io/exampleproject/job-wq-2
    Port:
    Environment:        <none>
    Mounts:             <none>
  Volumes:              <none>
Events:
  FirstSeen    LastSeen    Count    From            SubobjectPath    Type        Reason            Message
  ---------    --------    -----    ----            -------------    --------    ------            -------
  33s          33s         1        {job-controller }                Normal      SuccessfulCreate  Created pod: job-wq-2-lglf8


$ kubectl logs pods/job-wq-2-7r7b2
Worker with sessionID: bbd72d0a-9e5c-4dd6-abf6-416cc267991f
Initial queue state: empty=False
Working on banana
Working on date
Working on lemon
```


您可以看到，其中的一个 pod 处理了若干个工作单元。


## 其它


如果您不方便运行一个队列服务或者修改您的容器用于运行一个工作队列，您可以考虑其它的 [job 模式](/docs/concepts/jobs/run-to-completion-finite-workloads/#job-patterns)。


如果您有连续的后台处理业务，那么可以考虑使用 `replicationController` 来运行您的后台业务，和运行一个类似 [https://github.com/resque/resque](https://github.com/resque/resque) 的后台处理库。
