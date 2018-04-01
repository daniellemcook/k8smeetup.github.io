---
title: Kubernetes API 概念
approvers:
- bgrant0607
- smarterclayton
- lavalamp
- liggitt
cn-approvers:
- chentao1596
---


{% capture overview %}

本页介绍了 Kubernetes API 中的常用概念。
{% endcapture %}

{% capture body %}

Kubernetes API 是通过 HTTP 提供的基于资源（REST）的编程接口。它支持通过标准 HTTP 动词（POST，PUT，PATCH，DELETE，GET）检索、创建、更新和删除主资源，包括允许细粒度授权的许多对象的附加子资源（例如将 pod 绑定到节点）并且可以以不同的表示方式接受和服务这些资源，以方便或有效地使用。它还支持通过 "watches"  和一致列表对资源进行有效的更改通知，以允许其他组件有效缓存和同步资源状态。


## 标准 API 术语


大多数 Kubernetes API 资源类型都是 “对象” - 它们表示集群中概念的具体实例，如 pod 或命名空间。较少数量的 API 资源类型是 “虚拟” - 它们通常表示操作而不是对象，例如权限检查（使用带有 JSON 编码的 `SubjectAccessReview` 主体 POST 到 `subjectaccessreviews` 资源）。所有对象都有唯一的名称以允许幂等创建和检索，但如果虚拟资源类型不可检索或不依赖幂等性，则它们可能没有唯一名称。


Kubernetes 通常利用标准的 RESTful 术语来描述 API 概念：


* 一个 **资源类型** 是在 URL 中使用的名称（`pods`、`namespaces`、`services`）
* 所有资源类型在 JSON（它们的对象模式）中都有一个具体的表示形式，称为一种 **kind**
* 资源类型的实例列表被称为 **collection**
* 资源类型的单个实例称为 **resource**


所有资源类型以集群（`/apis/GROUP/VERSION/*`） 或命名空间（`/apis/GROUP/VERSION/namespaces/NAMESPACE/*`）为作用域。命名空间范围的资源类型在其命名空间被删除时将被删除，并且通过命名空间范围上的授权检查来控制对该资源类型的访问。以下路径用于检索集合和资源：


* 集群范围的资源：
  * `GET /apis/GROUP/VERSION/RESOURCETYPE` - 返回资源类型的资源集合
  * `GET /apis/GROUP/VERSION/RESOURCETYPE/NAME` - 在资源类型下使用 NAME 返回资源

* 命名空间范围的资源：
  * `GET /apis/GROUP/VERSION/RESOURCETYPE` - 在所有命名空间中返回资源类型的所有实例的集合
  * `GET /apis/GROUP/VERSION/namespaces/NAMESPACE/RESOURCETYPE` - 返回 NAMESPACE 中资源类型的所有实例的集合
  * `GET /apis/GROUP/VERSION/namespaces/NAMESPACE/RESOURCETYPE/NAME` - 在 NAMESPACE 中用 NAME 返回资源类型的实例


由于命名空间是一个集群范围的资源类型，因此您可以检索所有命名空间的列表 `GET /api/v1/namespaces` 以及关于特定命名空间的详细信息 `GET /api/v1/namespaces/NAME`。


几乎所有的对象资源类型都支持标准的 HTTP动 词 - GET、POST、PUT、PATCH 和 DELETE。Kubernetes 使用术语 **list** 来描述返回一组资源，以区别于检索通常称为 **get** 的单个资源。


一些资源类型将具有一个或多个子资源，在资源下方表示为子路径：


* 集群范围的子资源：`GET /apis/GROUP/VERSION/RESOURCETYPE/NAME/SUBRESOURCE`
* 命名空间范围的子资源：`GET /apis/GROUP/VERSION/namespaces/NAMESPACE/RESOURCETYPE/NAME/SUBRESOURCE`


支持每个子资源的动词将根据对象而有所不同 - 请参阅 API 文档的更多信息。无法跨多个资源访问子资源 - 如果需要，通常会使用新的虚拟资源类型。


## 有效检测变化


要使客户端能够构建集群当前状态的模型，需要所有 Kubernetes 对象资源类型支持一致的列表和增量更改通知反馈（称为 **watch**）。每个 Kubernetes 对象都有一个 `resourceVersion` 字段，表示该资源存储在底层数据库中的版本。当检索资源集合（命名空间或集群作用域）时，服务器的响应将包含 `resourceVersion` 该值可用于启动针对服务器的监视。服务器将返回所提供的 `resourceVersion` 之后的所有更改（创建、删除和更新）。这允许客户端获取当前状态，然后观察更改而不丢失任何更新。如果客户端 watch 断开连接，它们可以从上次返回的 `resourceVersion` 处重新启动一个新 watch，或者执行一个新的收集请求，然后重新开始。


例如：


1. 列出给定命名空间中的所有 pod。

        GET /api/v1/namespaces/test/pods
        ---
        200 OK
        Content-Type: application/json
        {
          "kind": "PodList",
          "apiVersion": "v1",
          "metadata": {"resourceVersion":"10245"},
          "items": [...]
        }


2. 从资源版本 10245 开始，将任何创建、删除或更新的通知作为单独的 JSON 对象接收。

        GET /api/v1/namespaces/test/pods?watch=1&resourceVersion=10245
        ---
        200 OK
        Transfer-Encoding: chunked
        Content-Type: application/json
        {
          "type": "ADDED",
          "object": {"kind": "Pod", "apiVersion": "v1", "metadata": {"resourceVersion": "10596", ...}, ...}
        }
        {
          "type": "MODIFIED",
          "object": {"kind": "Pod", "apiVersion": "v1", "metadata": {"resourceVersion": "11020", ...}, ...}
        }
        ...


给定的 Kubernetes 服务器只会在有限的时间内保留历史变更列表。使用 etcd2 的旧集群最多可保留1000次更改。默认情况下，使用 etcd3 的较新集群会在最近 5 分钟内保留更改。当由于该资源的历史版本不可用而导致请求的监视操作失败时，客户端必须通过识别状态码 `410 Gone` 、清除他们的本地缓存、执行列表操作以及从该新列表操作返回的 `resourceVersion` 开始监视来处理该事件。大多数客户端库为此逻辑提供某种形式的标准工具。（在 Go 中，这称为一个 `Reflector`，位于 `k8s.io/client-go/cache` 包中。）


## 以块为单位检索较大的结果集


在大型集群上，检索某些资源类型的集合可能会导致非常大的响应，从而影响服务器和客户端。例如，一个集群可能有成千上万个 pod，每个 pod 使用 JSON 编码都有 1-2kb。跨所有命名空间检索所有 pod 可能会导致非常大的响应（10-20MB）并消耗大量服务器资源。从 Kubernetes 1.9 开始，服务器支持将单个大型收集请求分成许多小块的能力，同时保持总请求的一致性。每个块可以按顺序返回，这样可以减少请求的总大小，并允许面向用户的客户端逐步显示结果以提高响应能力。


要以区块方式检索单个列表，在收集请求上受支持支持两个新参数 `limit` 和 `continue` ，并且从列表的 `metadata` 字段中的所有列表操作返回一个新的字段 `continue`。客户端应该使用 `limit` 指定他们希望在每个块中接收的最大结果，并且如果集合中有更多的资源，服务器将返回结果中的 `limit` 资源，并包含一个 `continue` 值。然后，客户端可以在下一个请求中将这个 `continue` 值传递给服务器，指示服务器返回下一个结果块。通过不断继续，直到服务器返回一个空的 `continue` 值，客户端才能使用完整的结果集。


就像 watch 操作一样，如果不能返回更多的结果，`continue` 令牌将在短时间内过期（默认为5分钟），并返回 `410 Gone`。在这种情况下，客户端需要从最开始的地方就开始，或者省略 `limit` 参数。


例如，如果集群上有 1,253 个 pod，并且客户端一次想要接收 500 个 pod 的块，他们将按如下方式请求这些块：


1. 列出集群中的所有 pod，每次检索最多 500 个pod。

        GET /api/v1/pods?limit=500
        ---
        200 OK
        Content-Type: application/json
        {
          "kind": "PodList",
          "apiVersion": "v1",
          "metadata": {
            "resourceVersion":"10245",
            "continue": "ENCODED_CONTINUE_TOKEN",
            ...
          },
          "items": [...] // returns pods 1-500
        }


2. 继续前面的请求，检索下一组 500 个pod。

        GET /api/v1/pods?limit=500&continue=ENCODED_CONTINUE_TOKEN
        ---
        200 OK
        Content-Type: application/json
        {
          "kind": "PodList",
          "apiVersion": "v1",
          "metadata": {
            "resourceVersion":"10245",
            "continue": "ENCODED_CONTINUE_TOKEN_2",
            ...
          },
          "items": [...] // returns pods 501-1000
        }


3，继续前面的请求，检索最后 253 ​​个 pod。

        GET /api/v1/pods?limit=500&continue=ENCODED_CONTINUE_TOKEN_2
        ---
        200 OK
        Content-Type: application/json
        {
          "kind": "PodList",
          "apiVersion": "v1",
          "metadata": {
            "resourceVersion":"10245",
            "continue": "", // continue token is empty because we have reached the end of the list
            ...
          },
          "items": [...] // returns pods 1001-1253
        }


请注意，列表的 `resourceVersion` 在每个请求中都保持不变，这表明服务器正在向我们显示 pod 的一致的快照。在版本 `10245` 之后创建、更新或删除的 pod 将不会显示，除非用户在没有 `continue` 令牌的情况下提出 list 请求。这允许客户端将大请求分解成较小的块，然后在完整的集合上执行 watch 操作，而不会丢失任何更新。



## 资源的替代表示


默认情况下，Kubernetes 返回序列化为 JSON 的对象，内容类型为 `application/json`。这是 API 的默认序列化格式。但是，客户端可能会请求这些对象更高效的 Protobuf 描述，以便在规模上更好地实现性能。Kubernetes API 实现了标准的 HTTP 内容类型：传递带有 `GET` 调用的 `Accept` 报头将请求服务器返回所提供的内容类型中的对象，同时将 Protobuf 中的对象带着 `Content-Type` 头发送到服务器进行 `PUT` 或者 `POST` 调用。如果支持所请求的格式，服务器将返回一个 `Content-Type` 头，如果提供了无效的内容类型，则返回 `406 Not acceptable` 的错误。


请参阅 API 文档以获取每种 API 支持的内容类型列表。


例如：


1. 以 Protobuf 格式列出集群中的所有 Pod。

        GET /api/v1/pods
        Accept: application/vnd.kubernetes.protobuf
        ---
        200 OK
        Content-Type: application/vnd.kubernetes.protobuf
        ... binary encoded PodList object


2. 通过将 Protobuf 编码数据发送到服务器来创建一个 pod，但是请求使用 JSON 响应。

        POST /api/v1/namespaces/test/pods
        Content-Type: application/vnd.kubernetes.protobuf
        Accept: application/json
        ... binary encoded Pod object
        ---
        200 OK
        Content-Type: application/json
        {
          "kind": "Pod",
          "apiVersion": "v1",
          ...
        }


并非所有 API 资源类型都支持 Protobuf，特别是那些通过自定义资源定义或 API 扩展定义的 API。必须针对所有资源类型工作的客户端应在其 `Accept` 报头中指定多个内容类型以支持 JSON 回退：

```
Accept: application/vnd.kubernetes.protobuf, application/json
```


### Protobuf 编码


Kubernetes 使用信封包装器来编码 Protobuf 响应。该包装器以 4 字节的魔术数字开始，以帮助识别磁盘或 etcd 中的内容为 Protobuf（与 JSON 相反），然后是 Protobuf 编码的包装消息，该消息描述底层对象的编码和类型，然后包含该对象。

包装格式是：

```
A four byte magic number prefix:
  Bytes 0-3: "k8s\x00" [0x6b, 0x38, 0x73, 0x00]

An encoded Protobuf message with the following IDL:
  message Unknown {
    // typeMeta should have the string values for "kind" and "apiVersion" as set on the JSON object
    optional TypeMeta typeMeta = 1;

    // raw will hold the complete serialized object in protobuf. See the protobuf definitions in the client libraries for a given kind.
    optional bytes raw = 2;

    // contentEncoding is encoding used for the raw data. Unspecified means no encoding.
    optional string contentEncoding = 3;

    // contentType is the serialization method used to serialize 'raw'. Unspecified means application/vnd.kubernetes.protobuf and is usually
    // omitted.
    optional string contentType = 4;
  }

  message TypeMeta {
    // apiVersion is the group/version for this type
    optional string apiVersion = 1;
    // kind is the name of the object schema. A protobuf definition should exist for this object.
    optional string kind = 2;
  }
```


接收到 `application/vnd.kubernetes.protobuf` 与该预期前缀不匹配的客户端应拒绝该响应，因为将来的版本可能需要以不兼容的方式更改序列化格式，并且将通过更改前缀来实现。

{% endcapture %}

{% include templates/concept.md %}
