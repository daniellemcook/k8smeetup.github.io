---
cn-approvers:
- tianshapjq
approvers:
- dchen1107
- yifan-gu
title: 使用 rkt 时的已知问题
---



当使用 rkt 作为容器运行时（runtime）时，以下的特性要么尚未支持，要么存在巨大的缺陷。计划在未来的版本中增加对这些特性和其他特性的支持，包括与默认容器引擎合理的特性匹配。


## 不存在的宿主机卷路径


当挂载一个不存在的主机卷路径时，rkt 将会出错。但是在 Docker 运行时（runtime）中，将会在被引用路径下创建一个空的目录。


例如以下的 pod 示例将会引发错误：

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: mount-dne
  name: mount-dne
spec:
  volumes:
  - name: does-not-exist
    hostPath:
      path: /does/not/exist
  containers:
    - name: exit
      image: busybox
      command: ["sh", "-c", "ls /test; sleep 60"]
      volumeMounts:
      - mountPath: /test
        name: does-not-exist
```


同时需要注意，如果在容器的 volumeMounts 指定了 `subPath`，但是对应的卷中并不存在该 `subPath` 路径，则 pod 的执行同样也会失败。


## Kubectl attach


`kubectl attach` 不能在 rkt 作为容器运行时（runtime）时使用。
因此，将不支持 `kubectl run` 中的一些参数，包括：

* `--attach=true`
* `--leave-stdin-open=true`
* `--rm=true`


## kvm 和 fly stage1s 的端口转发


如果 pod 是通过 `stage1-kvm` 或者 `stage1-fly` 执行的，那么将不支持 `kubectl port-forward` 功能。


## 卷的重新标签


当前 rkt 只支持 *per-pod* 的卷重新标签。重新标签后，已挂载的卷将会被 pod 中的所有容器共享。目前还没有办法能够让重新标签的卷只被 pod 中的某一个容器或者部分子网访问。更详细的信息可见 [Kubernetes issue # 28187](https://github.com/kubernetes/kubernetes/issues/28187)。


## kubectl get logs


目前在 rktnetes 环境下，如果应用直接将日志写入到 `/dev/stdout` 那么通过 `kubectl get logs` 命令是无法获取到应用日志的。目前这些日志内容都被打印到节点的控制台。


## Init Containers


目前并不支持 [Init Containers](/docs/concepts/workloads/pods/init-containers)。


## 容器的回退失败重启


当前不支持在重启失败的容器时按指数退避。


## 对实验性的 NVIDIA GPU 支持


`--experimental-nvidia-gpus` 参数和相关的 [GPU features](https://git.k8s.io/community/contributors/design-proposals/gpu-support.md) 都不支持。


## QoS 级别


在 rkt 下，QoS 级别不像在 Docker 一样能够匹配容器的 `OOM Score`。


## HostPID 和 HostIPC 命名空间


目前不支持给 pod 设置 hostPID 或者 hostIPC 参数。


例如，以下的 pod 示例将不会正确运行：

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: host-ipc-pid
  name: host-ipc-pid
spec:
  hostIPC: true
  hostPID: true
  containers:
    ...
```


另外，如果通过 [stage1-fly](https://coreos.com/rkt/docs/latest/running-fly-stage1.html) 来运行 pod，那么 pod 将会运行在宿主机的命令空间。


## 更新容器镜像 （patch）


通过 patch 更新容器的镜像将会导致整个 pod 重启，不仅仅是被修改的容器。


## ImagePullPolicy 'Always'


如果容器的镜像拉取策略是 `Always`，那么就算镜像没有任何修改 rkt 仍然会从远端拉取镜像。
如果镜像很大那么会增加很大的延迟。
这个 issue 可以通过 rkt 的上游进行跟踪，地址在 [#2937](https://github.com/coreos/rkt/issues/2937)。
