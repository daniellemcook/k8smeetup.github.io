---
approvers:
- bgrant0607
- janetkuo
title: kubectl 使用约定
cn-approvers:
- chentao1596
---


* TOC
{:toc}


## 在可重用脚本中使用 `kubectl`


如果脚本中需要确定的输出，您应该：


* 请求一个面向机器的输出表单，如 `-o name`， `-o json`， `-o yaml`， `-o go-template` 或者 `-o jsonpath`
* 指定 `--output-version`，因为这些输出表单（不包括 `-o name`）使用特定的 API 版本输出资源
* 如果使用基于生成器的命令（如 `kubectl run` 或 `kubectl expose`），请指定 `--generator` 永久地锁定特定行为
* 不要依赖上下文、首选项或其他隐式状态


## 最佳实践

### `kubectl run`


为 `kubectl run` 满足类似代码的架构：


* 总是用一个特定于版本的标签来标记您的镜像，不要把标签移到新的版本。例如，使用 `:v1234`， `v1.2.3`， `r03062016-1-4`，而不是 `:latest`（想了解更多信息，请查看 [配置的最佳实践](/docs/concepts/configuration/overview/#container-images)）。
* 如果镜像是轻微参数化的，可以捕获已签入脚本中的参数，或者至少使用 `--record` 来用命令行注释创建的对象。
* 如果镜像是高度参数化的，那么一定要签入脚本。
* 如果需要某些通过 `kubectl run` 标志无法表达的功能，切换到签入源代码管理的配置文件。
* 固定一个确定的 [生成器](#generators) 版本，如 `kubectl run --generator=deployment/v1beta1`。


#### 生成器


`kubectl Run` 允许您生成以下资源（使用 `--generator` 标志）：


* Pod - 使用 `run-pod/v1`。
* Replication controller - 使用 `run/v1`。
* Deployment, 使用 `extensions/v1beta1` 终端 - 使用 `deployment/v1beta1`（默认）。
* Deployment, 使用 `apps/v1beta1` 终端 - 使用 `deployment/apps.v1beta1`（推荐）。
* Job - 使用 `job/v1`。
* CronJob - 使用 `batch/v1beta1` 终端 - 使用 `cronjob/v1beta1`（默认）。
* CronJob - 使用 `batch/v2alpha1` 终端 - 使用 `cronjob/v2alpha1`（已废弃）。


此外，如果没有指定生成器标志，其他标志将建议使用特定的生成器。下表显示了哪些标志强制使用特定的生成器，具体取决于集群版本：


|   生成的资源           | 1.4 及以后版本集群     | 1.3 版本集群          | 1.2 版本集群                               | 1.1 及更早版本集群                         |
|:----------------------:|------------------------|-----------------------|--------------------------------------------|--------------------------------------------|
| Pod                    | `--restart=Never`      | `--restart=Never`     | `--generator=run-pod/v1`                   | `--restart=OnFailure` OR `--restart=Never` |
| Replication Controller | `--generator=run/v1`   | `--generator=run/v1`  | `--generator=run/v1`                       | `--restart=Always`                         |
| Deployment             | `--restart=Always`     | `--restart=Always`    | `--restart=Always`                         | N/A                                        |
| Job                    | `--restart=OnFailure`  | `--restart=OnFailure` | `--restart=OnFailure` OR `--restart=Never` | N/A                                        |
| Cron Job               | `--schedule=<cron>`    | N/A                   | N/A                                        | N/A                                        |


请注意，只有当您没有指定任何标志时，这些标志才会使用默认生成器。这也意味着将其他标志跟 `--generator` 组合不会改变您指定的生成器。例如，在一个 1.4 版本的集群中，如果指定 `--restart=Always`，将会创建一个 Deployment；如果同时指定 `--restart=Always` 和 `--generator=run/v1`，则创建一个 Replication Controller。如果您想使用生成器进行特定的行为，这也会变得非常方便，即使将来默认生成器发生了更改，它的行为也不会变化。


最后，设置生成器的标志的顺序是：调度标志拥有最高优先级，然后是重新启动策略，最后是生成器本身。


如果对所创建的最终资源存在疑问，您可以始终使用 `--dry-run` 标志，它将提供要提交到集群的对象。


### `kubectl apply`


* 若要使用 `kubectl apply` 更新资源，请始终使用 `kubectl apply` 或 `kubectl create --save-config` 创建最初的资源。想了解具体的原因，请查看 [使用 kubectl apply 管理资源](/docs/concepts/cluster-administration/manage-deployment/#kubectl-apply)。
