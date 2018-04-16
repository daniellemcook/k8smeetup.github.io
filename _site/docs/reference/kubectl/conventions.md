---approvers:
- bgrant0607
- janetkuo
cn-approvers:
- lichuqiang
title: kubectl 使用规约
---


* TOC
{:toc}


## 在可复用的脚本中使用 `kubectl`

如果您需要在脚本中提供稳定的输出，您应该：


* 请求一种面向机器的输出格式，例如 `-o name`、 `-o json`、 `-o yaml`、 `-o go-template` 或 `-o jsonpath`

* 由于那些输出格式（`-o name` 除外）使用特定 API 版本对资源进行输出，所以需要指定 `--output-version`

* 如果使用基于 generator 的命令（如 `kubectl run` 或 `kubectl expose`），请指定 --generator 以永远和特定行为绑定。

* 不要依赖上下文、首选项或其他隐式声明。


## 最佳实践

### `kubectl run`


为使 `kubectl run` 满足基础设施即代码（infrastructure as code）的要求：


* 总是用一个版本特定的标签来标记您的镜像，不要把标签移到新的版本。 例如，使用 `:v1234`、`v1.2.3` 和 `r03062016-1-4` 而不是 `:latest`
（参考 [配置的最佳实践](/docs/concepts/configuration/overview/#container-images) 了解更多信息）。

* 如果镜像的参数化程度很低，在检入脚本中捕捉这些参数，或者至少使用命令行 `--record` 来注释创建的对象。

* 如果镜像的参数化程度很高，一定要在脚本中检查。

* 如果需要一些无法通过 `kubectl run` 参数表达的特性，切换到签入源代码控制的配置文件。

* 与特定的 [generator](#generators) 版本固定，例如 `kubectl run --generator=deployment/v1beta1`。

#### Generators


`kubectl run` 允许您生成以下资源（使用 `--generator` 参数）：


* Pod - 使用 `run-pod/v1`。

* Replication controller - 使用 `run/v1`。

* Deployment，使用 `extensions/v1beta1` 端点 - 使用 `deployment/v1beta1`（默认）。

* Deployment，使用 `apps/v1beta1` 端点 - 使用 `deployment/apps.v1beta1`（推荐）。

* Job - 使用 `job/v1`。

* CronJob - 使用 `batch/v1beta1` 端点 - 使用 `cronjob/v1beta1`（默认）。

* CronJob - 使用 `batch/v2alpha1` 端点 - 使用 `cronjob/v2alpha1`（弃用）。


此外，如果没有指定 generator 参数，其他参数会推荐使用特定的 generator。 
下面的表格展示了根据集群版本，哪些参数强制使用特定的 generator：


|   生成的资源            | 1.4 或更高版本的集群     | 1.3 版本的集群          | 1.2 版本的集群                              | 1.1 或更低版本的集群                         |
|:----------------------:|------------------------|-----------------------|--------------------------------------------|--------------------------------------------|
| Pod                    | `--restart=Never`      | `--restart=Never`     | `--generator=run-pod/v1`                   | `--restart=OnFailure` 或 `--restart=Never` |
| Replication Controller | `--generator=run/v1`   | `--generator=run/v1`  | `--generator=run/v1`                       | `--restart=Always`                         |
| Deployment             | `--restart=Always`     | `--restart=Always`    | `--restart=Always`                         | N/A                                        |
| Job                    | `--restart=OnFailure`  | `--restart=OnFailure` | `--restart=OnFailure` 或 `--restart=Never` | N/A                                        |
| Cron Job               | `--schedule=<cron>`    | N/A                   | N/A                                        | N/A                                        |


注意：这些参数只有在您没有指定任何参数的情况下，才会使用默认 generator。 这也意味着把
`--generator` 和其他参数组合到一起不会改变您指定的 generator。 例如在 1.4 版本的集群中，
如果您指定了 `--restart=Always`，将会创建一个 Deployment，如果您指定了 `--restart=Always`
和 `--generator=run/v1`，将会创建一个 Replication Controller。

这在您想要将 generator 与特定行为固定的时候非常方便，即使将来默认 generator 发生变化。


最后，这些参数设置 generator 的顺序为：schedule 参数具有最高的优先级，然后是 restart 策略，
最后是 generator 本身。


如果对最终创建的资源有疑问，您可以始终使用 `--dry-run` 参数，它会提供要提交给集群的对象。


### `kubectl apply`


* 为使用 `kubectl apply` 来更新资源，最初总是使用 `kubectl apply` 或 `--save-config` 来创建资源。 查看 [使用 kubectl apply 管理资源](/docs/concepts/cluster-administration/manage-deployment/#kubectl-apply) 来了解背后的原因。
