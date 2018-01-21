---
assignees:
- erictune
- soltysh
cn-approvers:
- linyouchong
title: Jobs - Run to Completion
redirect_from:
- "/docs/concepts/jobs/run-to-completion-finite-workloads/"
- "/docs/concepts/jobs/run-to-completion-finite-workloads.html"
- "/docs/user-guide/jobs/"
- "/docs/user-guide/jobs.html"
---


* TOC
{:toc}


## 什么是 Job


一个 _Job_ 创建一个或多个 Pod，并确保存在指定数量的 Pod 能够成功终止。当一个 Pod 成功终止，_Job_ 对此进行跟踪记录。 当成功终止的次数达到一个指定数量时，Job 本身就完成了。 删除 Job 将清理它所创建的 Pod。


一个简单的例子是创建一个 Job 对象，以便让一个 Pod 能可靠地完成运行。如果第一个 Pod 运行失败或被删除（例如，由于节点硬件故障或节点重新启动），Job 对象将启动一个新的 Pod。


Job 也可以用于并行运行多个 Pod。


## 运行一个 Job 示例


这是一个示例 Job 配置。 它计算 π 值到第 2000 位并打印出来。大约需要 10 秒才能完成。

{% include code.html language="yaml" file="job.yaml" ghlink="/docs/concepts/workloads/controllers/job.yaml" %}


要运行示例 Job，先下载示例文件然后运行以下命令：

```shell
$ kubectl create -f ./job.yaml
job "pi" created
```


使用以下命令检查 Job 的状态：

```shell
$ kubectl describe jobs/pi
Name:             pi
Namespace:        default
Image(s):         perl
Selector:         controller-uid=b1db589a-2c8d-11e6-b324-0209dc45a495
Parallelism:      1
Completions:      1
Start Time:       Tue, 07 Jun 2016 10:56:16 +0200
Labels:           controller-uid=b1db589a-2c8d-11e6-b324-0209dc45a495,job-name=pi
Pods Statuses:    0 Running / 1 Succeeded / 0 Failed
No volumes.
Events:
  FirstSeen    LastSeen    Count    From            SubobjectPath    Type        Reason            Message
  ---------    --------    -----    ----            -------------    --------    ------            -------
  1m           1m          1        {job-controller }                Normal      SuccessfulCreate  Created pod: pi-dtn4q
```


查看 Job 的已完成的 Pod，使用 `kubectl get pod --show-all`。 `--show-all` 会显示已完成的 Pod。


以机器可读形式列出属于 Job 的所有 Pod，可以使用如下命令：

```shell
$ pods=$(kubectl get pods  --show-all --selector=job-name=pi --output=jsonpath={.items..metadata.name})
echo $pods
pi-aiw0a
```


在这里，选择器与 Job 的选择器相同。 `--output = jsonpath` 选项指定一个表达式，表示只获取返回列表中的每个 Pod 的名称。


查看其中一个 Pod 的标准输出

```shell
$ kubectl logs $pods
3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315155748572424541506959508295331168617278558890750983817546374649393192550604009277016711390098488240128583616035637076601047101819429555961989467678374494482553797747268471040475346462080466842590694912933136770289891521047521620569660240580381501935112533824300355876402474964732639141992726042699227967823547816360093417216412199245863150302861829745557067498385054945885869269956909272107975093029553211653449872027559602364806654991198818347977535663698074265425278625518184175746728909777727938000816470600161452491921732172147723501414419735685481613611573525521334757418494684385233239073941433345477624168625189835694855620992192221842725502542568876717904946016534668049886272327917860857843838279679766814541009538837863609506800642251252051173929848960841284886269456042419652850222106611863067442786220391949450471237137869609563643719172874677646575739624138908658326459958133904780275901
```


## 编写 Job 配置


与所有其他 Kubernetes 配置一样，Job 需要 `apiVersion` 、`kind` 和 `metadata` 字段。 有关使用配置文件的通用信息，请查看 [这里](/docs/user-guide/simple-yaml)、
[这里](/docs/user-guide/configuring-containers) 和 [这里](/docs/user-guide/working-with-resources)。


Job 还需要 [`.spec` section](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status) 字段。


### Pod 模板


`.spec.template` 是 `.spec` 的唯一的必填字段


`.spec.template` 是一个 [pod 模板](/docs/user-guide/replication-controller/#pod-template)。它与 [pod](/docs/user-guide/pods) 的语法几乎完全一样，除了它是内嵌字段而且没有 `apiVersion` 或 `kind` 字段。


除了 Pod 必需字段外，Job 中的 Pod 模板必须指定合适的标签（请参阅 [pod selector](#pod-selector)) 和合适的重启策略。


[`RestartPolicy`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) 只能等于 `Never` 或 `OnFailure`。


### Pod 选择器


`.spec.selector` 字段是可选的。在几乎所有情况下，您都不应该指定它。查看章节 [定义您自己的 Pod 选择器](#定义您自己的-pod-选择器)。



### 并行 Job


有三种主要类型的 Job：


1. 非并行 Job
  - 通常只启动一个 Pod ，除非 Pod 运行失败了。
  - Pod 一旦成功终止，任务就完成了。

1. 具有 *固定完成数量* 的并行 Job：
  - 为 `.spec.completions` 指定一个非零的正整数值
  - 当 1 到 `.spec.completions` 范围内的每个值都有一个成功的 pod 时，Job 就完成了
  - **尚未实现：** 每个 Pod 在 1 到 `.spec.completions` 范围内传递一个不同的索引。

1. 具有 *工作队列* 的并行 Job
  - 不要指定 `.spec.completions`，`.spec.parallelism` 保持默认值
  - Pod 必须与自己或外部服务协调，以确定每个 Pod 上每个应该工作的内容
  - 每个 Pod 独立地能够确定是否所有的同伴都已完成工作，从而完成整个工作。
  - 当 _任一_ Pod 成功终止时，不会再创建新的 Pod。
  - 一旦至少一个 Pod 已经成功终止，所有 Pod 都会被终止，那么 Job 就成功完成。
  - 一旦任何 Pod 成功退出，其他 Pod 就不应该继续做任何工作或写任何输出。 他们都应该处于退出的过程。


对于非并行 Job，您可以不设置 `.spec.completions` 和 `.spec.parallelism`。 两者都未设置时，默认都为1。


对于具有固定完成数量的 Job，您应该将 `.spec.completions` 设置为所需完成的数量。您可以设置 `.spec.parallelism`，或者不设置它，它默认为 1。


对于具有工作队列的 Job，您必须保留 `.spec.completions` 未设置，并将 `.spec.parallelism` 设置为非负整数。


有关如何使用不同类型 Job 的更多信息，请参阅 [Job 模式](#job-模式) 部分。



#### 控制并行度


请求的并行度（`.spec.parallelism`）可以设置为任何非负整数值。如果未指定，则默认为 1。如果将其指定为 0，则 Job 将被暂停，直到这个值被增加。


一个 Job 可以通过 `kubectl scale` 命令来扩容。 例如，下面的命令将一个名为 `myjob` 的 Job 的 `.spec.parallelism` 设置为 10：

```shell
$ kubectl scale  --replicas=$N jobs/myjob
job "myjob" scaled
```


您也可以使用 Job 资源的 `scale` 子资源。


实际的并行度（在任何时刻运行的 Pod 数量）可能多于或少于请求的并行度，原因有很多种。


对于具有固定完成数的 Job，并行运行的实际数量不会超过待完成数量。 `.spec.parallelism` 的较高值被有效地忽略。
- 
- 对于具有工作队列的 Job，任一 Pod 成功后都不会启动新的 Pod，但剩余的 Pod 可以继续完成。

- 如果控制器没有时间做出反应。

- 如果控制器由于某种原因（缺少ResourceQuota，缺少权限等）而无法创建 Pod，那么可能 Pod 的数量比请求的少。

- 由于同一 Job 中过多的先前的 Pod 故障，控制器可能会限制新的 Pod 创建。

- 当一个 Pod 正常关闭时，停止过程需要一定的时间。


## 处理 Pod 和容器故障


Pod 中的容器可能由于多种原因而失败，例如因为其中的进程退出时出现非零的退出代码，或者容器因为超出内存限制而死亡等。如果发生这种情况，那么 `spec.template.spec.restartPolicy = "OnFailure"`，那么 Pod 停留在节点上，但 Container 会重新运行。因此，当程序在本地重新启动时需要处理这种情况，否则应指定 `.spec.template.spec.restartPolicy = "Never"`。查看 [pods-states](/docs/user-guide/pod-states) 了解有关 `restartPolicy` 的更多信息。


整个 Pod 也可能会失败，原因很多，例如当 Pod 从节点上被踢出（节点被升级，重新引导，删除等等），或者 Pod 的容器失败以及 `.spec.template.spec.restartPolicy = "Never"`。 当一个 Pod 失败时，Job 控制器启动一个新的 Pod。 因此，您的程序需要处理重新启动时的情况。 尤其需要处理上次运行造成的临时文件、锁、不完整的输出等。


请注意，即使您指定了 `.spec.parallelism = 1` 和 `.spec.completions = 1` 和 `.spec.template.spec.restartPolicy = "Never"`，有时同一程序还是可能会被启动两次。


如果您指定 `.spec.parallelism` 和 `.spec.completions` 都大于 1，那么可能有多个 Pod 同时在运行。 因此，您的 Pod 还必须能够容忍并发性。


## Job 终止和清理


Job 完成后，不会再创建更多的 Pod，但 Pod 也不会被删除。 因为它们被终止了，所以它们不会出现在 `kubectl get pod` 的输出中，但是它们会出现在 `kubectl get pods -a` 的输出中。 保留它们可让您能够查看已完成 Pod 的日志，以检查错误、警告或其他诊断输出。 Job 对象在完成后也会被保留，以便查看其状态。 注意到它们的状态后，用户可以删除旧的 Job。 使用 `kubectl`（例如 `kubectl delete jobs/pi` 或 `kubectl delete -f ./job.yaml` ）删除作业。 当您使用 `kubectl` 删除作业时，它所创建的所有 Pod 也会被删除。


如果一个 Job 的 Pod 反复失败，Job 就会不断地创建新的 Pod，默认情况下，不断重试是一个有用的模式。 如果Job 的 Pod 的外部依赖缺失（例如，网络存储卷上的输入文件不存在），则 Job 将不断尝试 Pod，在您解决了外部依赖问题后（例如，创建缺少的文件 ），Job 将会运行完成而不需要任何进一步的干预。


但是，如果您不想不断重试，您可以设定 Job 的最后期限。 通过将 Job 的 `spec.activeDeadlineSeconds` 字段设置为一定数量的秒数来实现这个功能。 该 Job 将处于 `reason: DeadlineExceeded` 状态。 不会再创建 Pod，现有的 Pod 也将被删除。

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-timeout
spec:
  activeDeadlineSeconds: 100
  template:
    metadata:
      name: pi
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```


请注意，Job 规格和 Job 中的模板规格都有相同名称的字段。请在 Job 规格上设置。


## Job 模式


Job 对象可用于支持 Pod 的可靠并行执行。 Job 对象的设计目的不是为了支持在科学计算中常见的密切通信的并行进程。 它支持并行处理一组独立但相关的 *工作项* 。这些可能是要发送的电子邮件，要渲染的帧，要转码的文件，要扫描的 NoSQL 数据库中的密钥范围等。


在一个复杂的系统中，可能有多个不同的工作项目组。 在这里，我们只考虑用户想要一起管理的一组工作项目 &mdash; 一个*批量工作*。


并行计算有几种不同的模式，每种模式都有优缺点。权衡是：


- 每个工作项目使用一个 Job 对象，与所有工作项目使用单个 Job 对象相比。 后者对于大量的工作项目更好。 前者会给用户和系统造成一些开销用于管理大量的 Job 对象。 另外，对于后者，可以使用 `kubectl scale` 命令轻松调整作业的资源使用情况（同时运行的 Pod 数量）。

- 创建的 Pod 数量等于工作项目数量，与每个 Pod 可以处理多个工作项目相比。前者通常需要对现有代码和容器进行较少的修改。 后者对于大量的工作项目更好，因为与前面类似的原因。

- 有几种方法使用了工作队列。 这需要运行一个队列服务，并修改现有的程序或容器，使其使用工作队列。其他方法更容易适应现有的容器化应用程序。



这里总结了权衡，第 2 到第 4 列对应于上述权衡。模式名称也链接到了例子和更详细的描述。


|                            模式                                   | 单个 Job 对象 | Pod 数量比工作项目数量少? | 使用未修改的App? |  使用 Kube 1.1? |
| -------------------------------------------------------------------- |:-----------------:|:---------------------------:|:-------------------:|:-------------------:|
| [Job 模板扩展](/docs/tasks/job/parallel-processing-expansion/)            |                   |                             |          ✓          |          ✓          |
| [每个 Pod 处理一个工作项的队列](/docs/tasks/job/coarse-parallel-processing-work-queue/)   |         ✓         |                             |      有时      |          ✓          |
| [可变 Pod 数量的队列](/docs/tasks/job/fine-parallel-processing-work-queue/)  |         ✓         |             ✓               |                     |          ✓          |
| 静态分配工作的 Job                           |         ✓         |                             |          ✓          |                     |


当用 `.spec.completions` 指定完成数时，Job 控制器创建的每个 Pod 都有一个相同的 [`spec`](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status)。 这意味着所有的 Pod 将具有相同的命令行和相同的镜像，相同的卷和（几乎）相同的环境变量。 这些模式安排 Pod 在不同的事情上工作的不同方式。


这个表格显示了每个模式的 `.spec.parallelism` 和 `.spec.completions` 所需的设置。这里 `W` 是指工作项目的数量。


|                             模式                                  | `.spec.completions` |  `.spec.parallelism` |
| -------------------------------------------------------------------- |:-------------------:|:--------------------:|
| [Job 模板扩展](/docs/tasks/job/parallel-processing-expansion/)           |          1          |     必须为 1      |
| [每个 Pod 处理一个工作项的队列](/docs/tasks/job/coarse-parallel-processing-work-queue/)   |          W          |        任意值           |
| [可变 Pod 数量的队列](/docs/tasks/job/fine-parallel-processing-work-queue/)  |          1          |        任意值           |
| 静态分配工作的 Job



## 高级用法


### 定义您自己的 Pod 选择器


通常，当您创建一个 Job 对象时，您不需要指定 `spec.selector`。系统默认逻辑在 Job 创建时添加该字段。它选择一个不会与其他任务重叠的选择器值。


但是，在某些情况下，您可能需要重写此自动设置的选择器。为此，您可以指定 Job 的 `spec.selector`。


在这样做时要非常小心。如果您指定的标签选择器不是与该 Job 相同的标签选择器，并且与不相关的标签匹配，则无关 Job 的标签可能会被删除，或者该 Job 可能会将其他标签作为完成标签，或者其中一个或两个 Job 可能会拒绝创建 Pod 或运行完成。 如果选择了非唯一的选择器，则其他控制器（例如，ReplicationController）及其 Pod 也可能以不可预知的方式运行。 指定 `spec.selector` 时，Kubernetes不会阻止您犯这个错误。


以下是您可能需要使用此功能的示例。


假设 Job `old` 已经在运行。 您希望已有的 Pod 继续运行，但您希望剩下的 Pod 创建时使用不同的 Pod 模板，并让 Job 使用新的名称。 您无法更新 Job，因为这些字段是不可更新的。因此，使用 `kubectl delete jobs/old --cascade=false` 删除 Job `old`，但让它的 Pod 继续运行。在删除 Job 之前，请记下它所使用的选择器：

```
kind: Job
metadata:
  name: old
  ...
spec:
  selector:
    matchLabels:
      job-uid: a8f3d00d-c6d2-11e5-9f87-42010af00002
  ...
```


然后您创建一个名字为 `new` 的新 Job，并且明确地指定了相同的选择器。由于现有的 Pod 具有标签 `job-uid=a8f3d00d-c6d2-11e5-9f87-42010af00002`，所以它们也被 Job `new` 所控制。


您需要在新 Job 中指定 `manualSelector: true`，因为您没有使用系统通常为您自动生成的选择器。

```
kind: Job
metadata:
  name: new
  ...
spec:
  manualSelector: true
  selector:
    matchLabels:
      job-uid: a8f3d00d-c6d2-11e5-9f87-42010af00002
  ...
```


新的 Job 本身将有一个不同于 `a8f3d00d-c6d2-11e5-9f87-42010af00002` 的 uid。 设置 `manualSelector: true` 告诉系统，您知道您在做什么，并允许这种不匹配。


## 可选方案


### 直接使用 Pod


当运行 Pod 的节点重启或失败时，Pod 将终止并不会重新启动。 但是，一个 Job 将创建新的 Pod 来取代终止的 Pod。 出于这个原因，我们建议您使用 Job 而不是直接使用 Pod，即使您的应用程序只需要一个 Pod。


### Replication Controller


Job 是对 [Replication Controllers](/docs/user-guide/replication-controller) 的补充。 Replication Controller 管理预期不会终止的 Pod（例如 web 服务器），而 Job 管理预期终止的 Pod（例如批处理作业）。


正如 [Pod生命周期](/docs/concepts/workloads/pods/pod-lifecycle/) 中所讨论的，`Job` 只适用于 `RestartPolicy` 等于 `OnFailure` 或 `Never` 的 Pod。 （注意：如果没有设置 `RestartPolicy` ，默认值是 `Always`。）


### 单个 Job 启动控制器 Pod


另一种模式是为单个 Job 创建一个 Pod，然后由其创建其他 Pod，作为这些 Pod 的一种自定义控制器。 这样可以提供最大的灵活性，但是开始使用起来可能有点复杂，并且与 Kubernetes 的集成度较低。


这种模式的一个例子就是一个 Job 启动了一个 Pod，这个 Pod 运行一个脚本，然后启动一个 Spark 主控制器（参见 [spark example](https://github.com/kubernetes/kubernetes/tree/{{page.githubbranch}}/examples/spark/README.md)），运行一个 sparkdriver，然后清理。


这种方法的一个优点是，整个过程得到了一个 Job 对象的完整保证，且完整控制了创建的 Pod 和分配工作给他们的方式。


## Cron Job


在 Kubernetes [1.4](https://github.com/kubernetes/kubernetes/pull/11980) 中提供了在指定的时间/日期（即 cron）创建 Job 的支持。 更多信息可在 [cron Job 文档](/docs/concepts/workloads/controllers/cron-jobs/) 中找到。
