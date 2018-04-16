---
title: Rancher 与 Ubuntu Kubernetes 集成
cn-approvers:
- chentao1596
---

{% capture overview %}

本文说明如何在标准的 Kubernetes 上部署 Rancher 2.0alpha。


这些步骤目前处于 alpha/测试阶段，很可能会发生变化。


有关此集成的原始文档可以在 [https://github.com/CalvinHartwell/canonical-kubernetes-rancher/](https://github.com/CalvinHartwell/canonical-kubernetes-rancher/) 上找到。

{% endcapture %}
{% capture prerequisites %}

要使用本指南，您必须拥有一个使用标准的 juju 部署的处于工作状态的 kubernetes 集群。


有关使用 juju 部署 Kubernetes 的完整说明，请访问 [https://kubernetes.io/docs/getting-started-guides/ubuntu/installation/](https://kubernetes.io/docs/getting-started-guides/ubuntu/installation/)。  
{% endcapture %}


{% capture steps %}

## 部署 Rancher


想要部署 Rancher，我们只需要在 Kubernetes 上运行 Rancher 容器工作负载。Rancher 通过 dockerhub（[https://hub.docker.com/r/rancher/server/tags/](https://hub.docker.com/r/rancher/server/tags/)）提供他们的容器，并且可以从互联网免费下载。


如果您正在运行自己的仓库或进行离线部署，则在继续之前，应该下载容器镜像并其推入私有仓库。


### 使用 nodeport 部署 Rancher


首先创建一个 yaml 文件，该文件定义了如何在 kubernetes 上部署 Rancher。将该文件保存为 cdk-rancher-nodeport.yaml：

```
 ---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: default
    namespace: default
roleRef:
   kind: ClusterRole
   name: cluster-admin
   apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: rancher
  name: rancher
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rancher
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: rancher
        ima: pod
    spec:
      containers:
      - image: rancher/server:preview
        imagePullPolicy: Always
        name: rancher
        ports:
        - containerPort: 80
        - containerPort: 443
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          timeoutSeconds: 30
        resources: {}
      restartPolicy: Always
      serviceAccountName: ""
status: {}
---
apiVersion: v1
kind: Service
metadata:
  name: rancher
  labels:
    app: rancher
spec:
  ports:
    - port: 443
      protocol: TCP
      targetPort: 443
  selector:
    app: rancher
---
apiVersion: v1
kind: Service
metadata: 
  name: rancher-nodeport
spec: 
  type: NodePort
  selector:
     app: rancher
  ports: 
  - name: rancher-api
    protocol: TCP
    nodePort: 30443
    port: 443
    targetPort: 443
```


一旦 kubectl 运行并处于工作状态，运行以下命令来部署 Rancher：

```
  kubectl apply -f cdk-rancher-nodeport.yaml
```


现在我们需要打开这个 nodeport，以便我们可以访问它。为此，我们可以使用 juju。我们需要为集群中的每个工作节点运行 open-port 命令。在 cdk-rancher-nodeport.yaml 文件中，nodeport 已设置为 30443。下面显示了如何在每个工作节点上打开端口：


```
   # 对集群中的每个 Kubernetes 工作节点重复此操作。
   juju run --unit kubernetes-worker/0 "open-port 30443"
   juju run --unit kubernetes-worker/1 "open-port 30443"
   juju run --unit kubernetes-worker/2 "open-port 30443"
```


现在可以通过工作节点 IP 或 DNS 条目在此端口上访问 Rancher（如果已创建它们）。通常建议您为集群中的每个工作节点创建一个 DNS 条目。例如，如果您有三个工作节点并且您拥有域 example.com，则可以创建三条 A 记录，每个集群中的每个工作节点都有一条记录。


由于创建 DNS 条目超出了本文的范围，因此我们将使用可免费获得的 xip.io 服务，该服务可返回 A 记录作为域名一部分的 IP 地址。例如，如果您有域 rancher.35.178.130.245.xip.io，则 xip.io 服务将自动将 IP 地址 35.178.130.245 作为 A 记录返回，这对于测试目的很有用。对于您的部署，IP 地址 35.178.130.245 应该替换为您的一个工作节点 IP 地址，您可以使用 Juju 或 AWS 找到该 IP 地址：


```
 calvinh@ubuntu-ws:~/Source/cdk-rancher$ juju status

# ... 输出省略. 

Unit                      Workload  Agent  Machine  Public address  Ports                     Message
easyrsa/0*                active    idle   0        35.178.118.232                            Certificate Authority connected.
etcd/0*                   active    idle   1        35.178.49.31    2379/tcp                  Healthy with 3 known peers
etcd/1                    active    idle   2        35.177.99.171   2379/tcp                  Healthy with 3 known peers
etcd/2                    active    idle   3        35.178.125.161  2379/tcp                  Healthy with 3 known peers
kubeapi-load-balancer/0*  active    idle   4        35.178.37.87    443/tcp                   Loadbalancer ready.
kubernetes-master/0*      active    idle   5        35.177.239.237  6443/tcp                  Kubernetes master running.
  flannel/0*              active    idle            35.177.239.237                            Flannel subnet 10.1.27.1/24
kubernetes-worker/0*      active    idle   6        35.178.130.245  80/tcp,443/tcp,30443/tcp  Kubernetes worker running.
  flannel/2               active    idle            35.178.130.245                            Flannel subnet 10.1.82.1/24
kubernetes-worker/1       active    idle   7        35.178.121.29   80/tcp,443/tcp,30443/tcp  Kubernetes worker running.
  flannel/3               active    idle            35.178.121.29                             Flannel subnet 10.1.66.1/24
kubernetes-worker/2       active    idle   8        35.177.144.76   80/tcp,443/tcp,30443/tcp  Kubernetes worker running.
  flannel/1               active    idle            35.177.144.76                        

# 注意上面例子中 kubernetes 工作节点的 IP 地址。您应该选择一个公网地址。
```


尝试使用 nodeport 和域名或 IP 地址在浏览器中打开 Rancher：


```
  # 将 IP 地址替换为您的 Kubernetes 工作节点之一，从 juju 状态命令中找到此地址。
  wget https://35.178.130.245.xip.io:30443 --no-check-certificate

  # 这应该也可以工作
  wget https://35.178.130.245:30443 --no-check-certificate 
```


如果您需要对 kubernetes 配置文件进行任何更改，请编辑 yaml 文件，然后再次使用 apply：

```
  kubectl apply -f cdk-rancher-nodeport.yaml
```


### 使用 ingress 规则部署 Rancher


也可以使用 ingress 规则来部署 Rancher。这还有另外一个好处，就是不需要在 Kubernetes 集群上打开额外的端口。首先创建一个 yaml 文件来描述名为 cdk-rancher-ingress.yaml 的部署，它应包含以下内容：

```
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: default
    namespace: default
roleRef:
   kind: ClusterRole
   name: cluster-admin
   apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: rancher
  name: rancher
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rancher
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: rancher
    spec:
      containers:
      - image: rancher/server:preview
        imagePullPolicy: Always
        name: rancher
        ports:
        - containerPort: 443
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          timeoutSeconds: 30
        resources: {}
      restartPolicy: Always
      serviceAccountName: ""
status: {}
---
apiVersion: v1
kind: Service
metadata: 
  name: rancher
  labels:
    app: rancher
spec: 
  ports: 
    - port: 443
      targetPort: 443
      protocol: TCP
  selector:
    app: rancher
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
 name: rancher
 annotations:
   kubernetes.io/tls-acme: "true"
   ingress.kubernetes.io/secure-backends: "true"
spec:
 tls:
   - hosts:
     - rancher.34.244.118.135.xip.io
 rules:
   - host: rancher.34.244.118.135.xip.io
     http:
       paths:
         - path: /
           backend: 
             serviceName: rancher
             servicePort: 443
```


通常建议您为集群中的每个工作节点创建一个 DNS 条目。例如，如果您有三个工作节点并且您拥有域 example.com，则可以创建三条 A 记录，每个集群中的每个工作人员都有一条记录。


由于创建 DNS 条目超出了本教程的范围，因此我们将使用可免费获得的 xip.io 服务，该服务可以返回 A 记录作为域名一部分的 IP 地址。例如，如果您有域 rancher.35.178.130.245.xip.io，则 xip.io 服务将自动将 IP 地址 35.178.130.245 作为 A 记录返回，这对于测试目的很有用。


对于您的部署，IP 地址 35.178.130.245 应该替换为您的一个工作节点 IP 地址，您可以使用 Juju 或 AWS 找到该 IP 地址：


```
 calvinh@ubuntu-ws:~/Source/cdk-rancher$ juju status

# ... 输出省略。

Unit                      Workload  Agent  Machine  Public address  Ports                     Message
easyrsa/0*                active    idle   0        35.178.118.232                            Certificate Authority connected.
etcd/0*                   active    idle   1        35.178.49.31    2379/tcp                  Healthy with 3 known peers
etcd/1                    active    idle   2        35.177.99.171   2379/tcp                  Healthy with 3 known peers
etcd/2                    active    idle   3        35.178.125.161  2379/tcp                  Healthy with 3 known peers
kubeapi-load-balancer/0*  active    idle   4        35.178.37.87    443/tcp                   Loadbalancer ready.
kubernetes-master/0*      active    idle   5        35.177.239.237  6443/tcp                  Kubernetes master running.
  flannel/0*              active    idle            35.177.239.237                            Flannel subnet 10.1.27.1/24
kubernetes-worker/0*      active    idle   6        35.178.130.245  80/tcp,443/tcp,30443/tcp  Kubernetes worker running.
  flannel/2               active    idle            35.178.130.245                            Flannel subnet 10.1.82.1/24
kubernetes-worker/1       active    idle   7        35.178.121.29   80/tcp,443/tcp,30443/tcp  Kubernetes worker running.
  flannel/3               active    idle            35.178.121.29                             Flannel subnet 10.1.66.1/24
kubernetes-worker/2       active    idle   8        35.177.144.76   80/tcp,443/tcp,30443/tcp  Kubernetes worker running.
  flannel/1               active    idle            35.177.144.76

# 注意上面例子中 kubernetes 工作节点的 IP 地址。您应该选择一个公网地址。
```


查看上述 juju 状态的输出，公共地址（35.178.130.245）可用于创建 xip.io DNS 条目（rancher.35.178.130.245.xip.io），该条目应放置在 cdk-rancher- ingress.yaml 文件。您也可以创建自己的 DNS 条目，只要它解析为每个工作节点或其中一个工作节点即可正常工作：


```
  # 文件中，xip.io 域应该出现在两个地方，都需要修改
  cat cdk-rancher-ingress.yaml | grep xip.io
  - host: rancher.35.178.130.245.xip.io
```


一旦编辑了 ingress 规则以映射您的 DNS 条目，请运行 kubectl apply -f cdk-rancher-ingress.yaml 来部署 Kubernetes：

```
 kubectl apply -f cdk-rancher-ingress.yaml
```


现在可以通过工作节点 IP 或 DNS 条目在常规 443 上访问 Rancher（如果已创建它们）。尝试在浏览器中打开它：


```
  # 将 IP 地址替换为您的 Kubernetes 工作节点之一，从 juju 状态命令中找到此地址。
  wget https://35.178.130.245.xip.io:443 --no-check-certificate
```


如果您需要对 kubernetes 配置文件进行任何更改，请编辑 yaml 文件，然后再次使用 apply：

```
  kubectl apply -f cdk-rancher-ingress.yaml
```


### 移除 Rancher


您可以使用 kubectl 从集群中删除 Rancher。在 Kubernetes 中删除结构与创建它们一样简单：


```
  # 使用 nodeport 示例（如果使用 ingress 示例，请修改文件名）
  kubectl delete -f cdk-rancher-nodeport.yaml
```
{% endcapture %}

{% include templates/task.md %}
