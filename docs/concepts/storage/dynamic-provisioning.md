---
approvers:
- saad-ali
cn-approvers:
- lichuqiang
title: 动态卷供应（Provision）
---


{% capture overview %}


动态卷供应允许按需创建存储卷。 如果没有动态供应，集群管理员必须手动调用其云服务或存储提供商（的接口）来创建新的存储卷，
然后在 Kubernetes 中创建 [`PersistentVolume` 对象](/docs/concepts/storage/persistent-volumes/)
来代表这些卷。 动态供应特性使得集群管理员不必预先准备存储卷。 相反，它在用户请求时自动提供存储。

{% endcapture %}

{:toc}

{% capture body %}


## 背景

动态卷供应的实现基于 `storage.k8s.io` API 组 中的 `StorageClass` API 对象。
集群管理员可以根据需要，定义任意数量的 `StorageClass` 对象，
每个 `StorageClass` 对象中指定一个 *卷插件*（又名*提供商*），
以及动态供应时传递到提供商的一系列参数。


集群管理员可以在集群中定义和暴露多种不同的存储（来自相同或不同的存储系统），
每种存储都有自定义的参数集。这样的设计也保证了最终用户不必关心存储供应的复杂性和细微差别，
但仍有能力从多个存储选项中进行选择。


更多关于存储类（storage class）的信息请参考
[这里](/docs/concepts/storage/persistent-volumes/#storageclasses)。


## 启用动态供应

为启用动态供应，集群管理员需要预先为用户创建一个或多个 StorageClass 对象。

StorageClass 对象定义了调用动态供应时所使用的提供商和所传入的参数。
下面的声明创建了一个名为 “slow” 的存储类，它提供类似标准磁盘的持久化磁盘。

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
```



下面的声明创建了一个名为 “fast” 的存储类，它提供类似 SSD 的持久化磁盘。

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```


## 使用动态供应

用户通过在 `PersistentVolumeClaim` 中包含存储类来请求使用动态供应的存储。
在 Kubernetes 1.6 版本之前，存储类信息通过
`volume.beta.kubernetes.io/storage-class` 注解来指定。 然而，从 1.6 版本开始，
该注解不再推荐使用。 用户现在可以并且应该使用 `PersistentVolumeClaim` 对象的
`storageClassName` 字段来代替该注解。 该字段的值须与管理员配置的
`StorageClass` 名称相匹配 (参考 [这里](#启用动态供应))。


例如，为选择 “fast” 存储类，用户应该创建以下 `PersistentVolumeClaim`：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 30Gi
```


该 claim 使得一个 类似 SSD 的持久磁盘被自动提供。 当 claim 被删除时，卷也被相应销毁。


## 默认行为

动态供应可以按照这样的方式启用： claim 未指定存储类的情况下，会动态配置（为默认存储类）。
群集管理员可以通过以下方式启用此行为：


- 将一个 `StorageClass` 对象标记为 *default*，
- 保证 [`DefaultStorageClass` 准入控制器](/docs/admin/admission-controllers/#defaultstorageclass)
  在 API 服务器上启用。


管理员可以通过为特定的 `StorageClass` 添加 `storageclass.kubernetes.io/is-default-class`
注解，将其标记为默认存储类。 当集群中存在一个默认 `StorageClass`，且用户创建了一个未指定
`storageClassName` 的 `PersistentVolumeClaim` 时，`DefaultStorageClass` 准入控制器自动为其添加 `storageClassName` 字段，字段指向默认存储类。


注意集群中最多只能有一个 *default* 存储类，否则将无法创建未明确指定 `storageClassName`
的 `PersistentVolumeClaim`。

{% endcapture %}

{% include templates/concept.md %}
