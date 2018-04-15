---
title: 参与 SIG-DOCS
cn-approvers:
- chentao1596
---


{% capture overview %}


SIG-DOCS 是 Kubernetes 项目中的一个 [特殊兴趣小组](https://github.com/kubernetes/community/blob/master/sig-list.md), 重点是编写、更新和维护 Kubernetes 的文档。

{% endcapture %}

{% capture body %}


SIG Docs 欢迎来自所有贡献者的内容和审核。任何人都可以打开一个拉取请求（PR），欢迎任何人评论内容或正在进行中的 PR。


在 Kubernetes 项目中，您也可以成为成员、审核者或批准人。这些角色在批准和提交更改时赋予其他特权和责任。查看 [社区成员关系](https://github.com/kubernetes/community/blob/master/community-membership.md) 了解有关 Kubernetes 社区中成员运作的更多信息。


## 角色和责任


自动读取 `/hold`、`/lgtm` 和 `/approve` 评论，并在 PR 上设置标签。当一个 PR 有 `lgtm` 和 `approve` 并且没有任何 `hold` 标签，PR 自动合并。Kubernetes 组织的 SIG Docs 成员、审核人和批准人可以添加评论来控制合并自动化。


- 成员


[Kubernetes 组织](https://github.com/kubernetes) 的任何成员都可以审查 PR，SIG Docs 团队成员经常要求其他 SIG 成员的评论以获得技术准确性。不管是 Kubernetes 组织的什么成员，SIG Docs 都欢迎您的评论和反馈。您可以通过添加 `/lgtm` 对 PR 的评论来表明您的批准。


- 审核人


审核人是审阅文档 PR 的人。


自动指派审核人给 PR，贡献者也可以通过对 PR 进行评论请求审阅：`/assign [@_github_handle]`。要表明 PR 不需要进一步更改，审核人应该向 PR 添加评论 `/lgtm`。审核人通过 `/lgtm` 表明技术的准确性。


审核人可以添加 `/hold` 评论以防止合并 PR。另一位审核人或批准人可以通过评论删除暂停：`/hold cancel`。


当审核人被分配了审核请求时，他并不是唯一的责任人，任何其他审核人员也可以根据该 PR 提供他们的意见。如果需要审核人，人们普遍认为，PR 将留给审核人去做编辑传递的内容。如果 PR 作者或 SIG Docs 维护人员要求审核，请不要合并或关闭 PR，直到被请求的审核人完成审核。


- 批准人


批准人有能力合并 PR。


批准人可以通过对 PR 的评论表明他们的批准：`/approve`。批准人使用 `/approve` 评论表明编辑批准。


批准人可以添加 `/hold` 注释以防止合并请求。另一位审核人或批准人可以通过评论删除暂停：`/hold cancel`。


如果提议的更改显得微不足道和/或非常容易理解，批准人可能会跳过对小 PR 的进一步审核。批准人可以在 PR 评论中表明 `/lgtm` 或者 `/approve`，并合并 PR，所有的 PR 要求至少有一个批准人提供他们的投票，以便 PR 被合并。


**注意：** 批准人使用评论时存在特殊情况：`/lgtm`。在这些情况下，自动化将同时添加 `lgtm` 和 `approve` 标签，从而跳过任何进一步的审查。
+{: .note }


对于不需要审核的 PR（拼写错误或其他微不足道的变化），批准人可以输入 `lgtm` 评论，表示无需进一步审核，并在批准合并时标记 PR。


### SIG Docs 中的团队和小组


您可以从 [社区 github 仓库获取 SIG Docs](https://github.com/kubernetes/community/tree/master/sig-docs) 的概述。SIG Docs 小组在 Github 上定义了两个团队：
 - [@kubernetes/sig-docs-maintainers](https://github.com/orgs/kubernetes/teams/sig-docs-maintainers)
 - [@kubernetes/sig-docs-pr-reviews](https://github.com/orgs/kubernetes/teams/sig-docs-pr-reviews)


这些组织维护 [Kubernetes website 仓库](https://github.com/kubernetes/website)，该仓库包含在此网站上托管的内容。两者都可以在 github 评论中使用 `@name` 引用，以便与该组中的每个人进行交流。


这些团队有人员重叠，但不完全匹配自动化工具使用的组。为了分配 issue、PR 并支持 PR 批准，自动化使用来自 OWNERS 文件的信息。


要自愿作为审核人或批准人，请提出 PR，并将您的 Github 句柄添加到 [OWNERS 文件](https://github.com/kubernetes/community/blob/master/contributors/devel/owners.md) 中的相关部分。


**注意：** 审核人和批准人必须符合参与要求。有关更多信息，请参阅 [Kubernetes 社区](https://github.com/kubernetes/community/blob/master/community-membership.md#membership) 仓库。
{: .note }


[OWNERS](https://github.com/kubernetes/community/blob/master/contributors/devel/owners.md) 的文档说明了如何维护每个启用它的仓库的 OWNERS。


[Kubernetes website 仓库](https://github.com/kubernetes/website) 有两个自动化（prow） [插件启用](https://github.com/kubernetes/test-infra/blob/master/prow/plugins.yaml#L210)：
- blunderbuss
- approve


这两个插件使用我们的仓库中的 [OWNERS](https://github.com/kubernetes/website/blob/master/OWNERS) 和 [OWNERS_ALIAS](https://github.com/kubernetes/website/blob/master/OWNERS_ALIAS) 文件进行配置。

{% endcapture %}

{% capture whatsnext %}

有关贡献 Kubernetes 文档的更多信息，请参阅：


* 查看SIG Docs [样式指南](/docs/home/contribute/style-guide/)。
* 了解如何 [进行文档更改](/docs/home/contribute/stage-documentation-changes/)。
* 了解如何 [撰写新话题](/docs/home/contribute/write-new-topic/)。
* 了解如何 [使用页面模板](/docs/home/contribute/page-templates/)。
* 了解有关 [模拟您的更改](/docs/home/contribute/stage-documentation-changes/)。
* 了解如何 [创建 PR](/docs/home/contribute/create-pull-request/)。
* 如何生成文档：
  * 了解如何 [为 Kubernetes Federation API 生成参考文档](/docs/home/contribute/generated-reference/federation-api/)
  * 了解如何 [为 kubectl 命令生成参考文档](/docs/home/contribute/generated-reference/kubectl/)
  * 了解如何 [为 Kubernetes API 生成参考文档](/docs/home/contribute/generated-reference/kubernetes-api/)
  * 了解如何 [为 Kubernetes 组件和工具生成参考页](/docs/home/contribute/generated-reference/kubernetes-components/)
{% endcapture %}

{% include templates/concept.md %}
