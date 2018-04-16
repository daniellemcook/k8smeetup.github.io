# ConfigMap 示例




## 第0步：先决条件


本例假定您已经安装好一个 Kubernetes 集群并正在运行，并在您的路径中安装了 `kubectl` 命令行工具。 请访问 [选择正确的解决方案](/docs/setup/pick-right-solution/) 查看适合您的平台的安装说明。


## 第1步：创建 ConfigMap


一个 ConfigMap 包含一组命名的字符串


使用 [`configmap.yaml`](configmap.yaml) 文件创建一个 ConfigMap ：

```shell
$ kubectl create -f docs/user-guide/configmap/configmap.yaml
```


您可以使用 `kubectl` 查看关于这个 ConfigMap 的信息：

```shell
$ kubectl get configmap
NAME                   DATA      AGE
test-configmap         2         6s

$ kubectl describe configMap test-configmap
Name:          test-configmap
Labels:        <none>
Annotations:   <none>

Data
====
data-1: 7 bytes
data-2: 7 bytes
```


使用 `kubectl get` 查看 ConfigMap 中存储的键值对：

```shell
$ kubectl get configmaps test-configmap -o yaml
apiVersion: v1
data:
  data-1: value-1
  data-2: value-2
kind: ConfigMap
metadata:
  creationTimestamp: 2016-02-18T20:28:50Z
  name: test-configmap
  namespace: default
  resourceVersion: "1090"
  selfLink: /api/v1/namespaces/default/configmaps/test-configmap
  uid: 384bd365-d67e-11e5-8cd0-68f728db1985
```


## 第2步：创建一个 pod 并在其环境变量中使用 configMap


使用 [`env-pod.yaml`](env-pod.yaml) 文件创建一个 Pod 并在其环境变量中使用 ConfigMap ：

```shell
$ kubectl create -f docs/user-guide/configmap/env-pod.yaml
```


这个 pod 运行 `env` 命令来展示容器中的环境变量：

```shell
$ kubectl logs config-env-test-pod | grep KUBE_CONFIG
KUBE_CONFIG_1=value-1
KUBE_CONFIG_2=value-2
```


## 第3步：创建一个 pod 并使用 ConfigMap 设置其命令行


使用 [`command-pod.yaml`](command-pod.yaml) 文件创建一个 Pod 并将 ConfigMap 的键注入容器的命令行：

```shell
$ kubectl create -f docs/user-guide/configmap/command-pod.yaml
```


这个 pod 运行 `echo` 命令来展示这些键对应的值：

```shell
$ kubectl logs config-cmd-test-pod
value-1 value-2
```


## 第4步：创建一个在 volume 中使用 configMap 的 pod


Pod 也可以在 volume 中使用 ConfigMap 。使用 [`volume-pod.yaml`](volume-pod.yaml) 文件来创建一个 Pod 并在其 volume 中使用 ConfigMap ：

```shell
$ kubectl create -f docs/user-guide/configmap/volume-pod.yaml
```


这个 pod 运行 `cat` 命令来打印 volume 中某个键对应的值：

```shell
$ kubectl logs config-volume-test-pod
value-1
```


或者您也可以使用 [`mount-file-pod.yaml`](mount-file-pod.yaml) 文件仅从 ConfigMap 中挂载一个文件，并保留 /etc 目录中其他原始内容。
