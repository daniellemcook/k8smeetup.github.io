---
title: 服务目录
approvers:
- chenopis
---


{% capture overview %}

*服务代理（Service Broker）* 是由 [开放服务代理 API 规范（Open Service Broker API spec）](https://github.com/openservicebrokerapi/servicebroker/blob/v2.13/spec.md)定义的一组托管服务的终结点（endpoint），由第三方提供并维护，其中的第三方可以是 AWS，GCP 或 Azure 等云服务提供商。


集群运维人员可以使用服务目录浏览其提供的托管服务列表，提供托管服务实例并与之绑定，以使其可以被 Kubernetes 集群中的应用程序使用。

{% endcapture %}


{% capture body %}

## 示例用例


应用开发者希望使用消息队列作为其在 Kubernetes 集群中运行的应用程序的一部分。但是，它们不想承受建立这种服务的开销，也不想自行管理。幸运的是，有一家云服务提供商通过它们的 *服务代理* 将消息队列作为 *托管服务* 提供。


集群运维人员通信，以此提供消息队列服务的实例并使其对 Kubernetes 中的应用程序可用。因此，应用开发者可以不用关心消息队列的实现细节，也不用对其进行管理。它们的应用程序可以简单的将其作为服务使用。


## 架构


服务目录使用  [开放服务代理 API](https://github.com/openservicebrokerapi/servicebroker) 与服务代理进行通信，并作为 Kubernetes API Server 的中介，以便协商首要规定（initial provisioning）并获取应用程序使用托管服务的必要凭据。


它被实现为一个扩展 API 服务和一个控制器管理器，使用 Etcd 作为存储。它还使用了 Kubernetes 1.7+ 版本中提供的 [aggregation layer](/docs/concepts/api-extension/apiserver-aggregation/) 来呈现其 API。

<br>


![服务目录架构](/images/docs/service-catalog-architecture.svg)



### API 资源


服务目录安装  `servicecatalog.k8s.io` API  并提供以下 Kubernetes 资源：


* `ClusterServiceBroker`：服务目录的集群内代表，封装了它的服务连接细节。集群运维人员创建和管理这些资源，并希望使用该代理服务在集群中提供新类型的托管服务。
* `ClusterServiceClass`：由特定服务代理提供的托管服务。当新的 `ClusterServiceBroker` 资源被添加到集群时，服务目录控制器将连接到服务代理以获取可用的托管服务列表。然后为每个托管服务创建对应的新 `ClusterServiceClass` 资源。
* `ClusterServicePlan`：托管服务的特定产品。例如托管服务可能有不同的计划可用，如免费版本和付费版本，或者可能有不同的配置选项，例如使用 SSD 存储或拥有更多资源。与 `ClusterServiceClass` 类似，当一个新的 `ClusterServiceBroker` 被添加到集群时，服务目录会为每个托管服务的每个可用服务计划（Service Plan）创建对应的新 `ClusterServicePlan` 资源。
* `ServiceInstance`：`ClusterServiceClass` 提供的示例。
  由集群运维人员创建，以使托管服务的特定实例可供一个或多个集群内应用程序使用。
  当创建一个新的 `ServiceInstance` 资源时，服务目录控制器将连接到相应的服务代理并指示它调配服务实例。
* `ServiceBinding`：`ServiceInstance` 的访问凭据。
  由希望其应用程序使用服务 `ServiceInstance` 的集群运维人员创建。
  创建之后，服务目录控制器将创建一个 Kubernetes `Secret`，其中包含服务实例的连接细节和凭据，可以挂载到 Pod 中。


### 认证


服务目录支持这些认证方法：

* 基础认证（用户名/密码）
* [OAuth 2.0 不记名令牌（Bearer Token）](https://tools.ietf.org/html/rfc6750)


## 使用方式


集群运维人员可以使用服务目录 API 资源来提供托管服务并使其在 Kubernetes 集群内可用。涉及的步骤有：

1. 列出服务代理提供的托管服务和服务计划。
2. 配置托管服务的新实例。
3. 绑定到托管服务，它将返回连接凭证。
4. 将连接凭证映射到应用程序中。


### 列出托管服务和服务计划


首先，集群运维人员在 `servicecatalog.k8s.io` 组内创建一个 `ClusterServiceBroker` 资源。此资源包含访问服务代理终结点所需的 URL 和连接详细信息。


这是一个 `ClusterServiceBroker` 资源的例子：

```yaml
apiVersion: servicecatalog.k8s.io/v1beta1
kind: ClusterServiceBroker
metadata:
  name: cloud-broker
spec:
  # Points to the endpoint of a Service Broker. (This example is not a working URL.)
  url:  https://servicebroker.somecloudprovider.com/v1alpha1/projects/service-catalog/brokers/default
  #####
  # Additional values can be added here, which may be used to communicate
  # with the Service Broker, such as bearer token info or a caBundle for TLS.
  #####
```


下面的顺序图展示了从一个服务代理列出可用托管服务和计划所有涉及的步骤：


![列出服务](/images/docs/service-catalog-list.svg){:height="80%" width="80%"}


1. 一旦 `ClusterServiceBroker` 资源被添加到了服务目录之后，将会触发一个到外部服务代理的 *List Services* 调用。
2. 服务代理返回可用的托管服务和服务计划列表，这些列表将本地缓存在 `ClusterServiceClass` 和 `ClusterServicePlan` 资源中。
3. 然后集群运维人员可以使用以下命令获取可用托管服务的列表：

        kubectl get clusterserviceclasses -o=custom-columns=SERVICE\ NAME:.metadata.name,EXTERNAL\ NAME:.spec.externalName


    它应该输出一个和以下格式类似的服务名称列表：

        SERVICE NAME                           EXTERNAL NAME
        4f6e6cf6-ffdd-425f-a2c7-3c9258ad2468   cloud-provider-service
        ...                                    ...


    他们还可以使用以下命令查看可用的服务计划：

        kubectl get clusterserviceplans -o=custom-columns=PLAN\ NAME:.metadata.name,EXTERNAL\ NAME:.spec.externalName


    它应该输出一个和以下格式类似的服务计划列表：

        PLAN NAME                              EXTERNAL NAME
        86064792-7ea2-467b-af93-ac9694d96d52   service-plan-name
        ...                                    ...



### 配置一个新实例


集群运维人员可以通过创建一个 `ServiceInstance` 资源来启动一个新实例的配置。

这是一个 `ServiceInstance` 资源的例子：

```yaml
apiVersion: servicecatalog.k8s.io/v1beta1
kind: ServiceInstance
metadata:
  name: cloud-queue-instance
  namespace: cloud-apps
spec:
  # References one of the previously returned services
  clusterServiceClassExternalName: cloud-provider-service
  clusterServicePlanExternalName: service-plan-name
  #####
  # Additional parameters can be added here, 
  # which may be used by the Service Broker.
  #####
```


以下顺序图展示了配置托管服务新实例所涉及的步骤：


![配置服务](/images/docs/service-catalog-provision.svg){:height="80%" width="80%"}


1. 当创建 `ServiceInstance` 资源时，服务目录将启动一个到外部服务代理的 *配置实例* 调用。
2. 服务代理创建一个托管服务的新实例并返回 HTTP 响应。
3. 然后集群运维人员可以检查实例的状态是否就绪。


### 绑定到托管服务


在设置新实例之后， 集群运维人员必须绑定到托管服务才能获取应用程序使用服务所需的连接凭据和服务账户的详细信息。该操作通过创建一个 `ServiceBinding` 资源完成。 


以下是 `ServiceBinding` 资源的示例：

```yaml
apiVersion: servicecatalog.k8s.io/v1beta1
kind: ServiceBinding
metadata:
  name: cloud-queue-binding
  namespace: cloud-apps
spec:
  instanceRef:
    name: cloud-queue-instance
  #####
  # Additional information can be added here, such as a secretName or 
  # service account parameters, which may be used by the Service Broker.
  #####
```

以下顺序图展示了绑定到托管服务实例的步骤：

![绑定到托管服务](/images/docs/service-catalog-bind.svg){:height="80%" width="80%"}


1. 在创建 `ServiceBinding` 之后，服务目录将进行一次到外部服务代理的 *Bind Instance*  调用。
2. 服务代理为相应服务账户启用应用权限/角色。
3. 服务代理返回连接和访问托管服务示例所需的信息。这是由提供商和服务特定的，故返回的信息可能因服务提供商和其托管服务而有所不同。


### 映射连接凭据

完成绑定之后的最后一步就是将连接凭据和服务特定的信息映射到应用程序中。
这些信息存储在 secret 中，集群中的应用程序可以访问并使用它们直接与托管服务进行连接。

<br>


![映射连接凭据](/images/docs/service-catalog-map.svg)


#### Pod 配置文件


执行此映射的一种方法是使用声明式 Pod 配置。

以下示例描述了如何将服务账户凭据映射到应用程序中。名为 `sa-key` 的密钥保存在一个名为 `provider-cloud-key` 的卷中，应用程序会将该卷挂载在 `/var/secrets/provider/key.json` 路径下。环境变量 `GOOGLE_APPLICATION_CREDENTIALS` 将映射为挂载文件的路径。

```yaml
...
    spec:
      volumes:
        - name: provider-cloud-key
          secret:
            secretName: sa-key
      containers:
...
          volumeMounts:
          - name: provider-cloud-key
            mountPath: /var/secrets/provider
          env:
          - name: PROVIDER_APPLICATION_CREDENTIALS
            value: "/var/secrets/provider/key.json"
```


以下示例描述了如何将 secret 值映射为应用程序的环境变量。在这个示例中，消息队列的主题名从 secret `provider-queue-credentials` 中名为 `topic` 的 key  项映射到环境变量 `TOPIC` 中。


```yaml
...
          env:
          - name: "TOPIC"
            valueFrom:
                secretKeyRef:
                   name: provider-queue-credentials
                   key: topic
```

{% endcapture %}


{% capture whatsnext %}

* 如果您熟悉 Helm Charts，您可以 [使用 Helm 将服务目录](/docs/tasks/service-catalog/install-service-catalog-using-helm/)  安装到 Kubernetes 集群中。或者，您可以 [使用 SC 工具安装服务目录](/docs/tasks/service-catalog/install-service-catalog-using-sc/)。
* 查看 [服务代理示例](https://github.com/openservicebrokerapi/servicebroker/blob/master/gettingStarted.md#sample-service-brokers)。
* 浏览 [kubernetes-incubator/service-catalog](https://github.com/kubernetes-incubator/service-catalog) 项目。
{% endcapture %}


{% include templates/concept.md %}
