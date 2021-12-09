## client-go源码阅读
### 概括
client-go 是用 Golang 语言编写的官方编程式交互客户端库，提供对 Kubernetes API server 服务的交互访问。

其源码目录结构如下：
1. discovery: 提供 DiscoveryClient 发现客户端。
2. dynamic: 提供 DynamicClient 动态客户端。
3. informers: 每种 K8S 资源的 Informer 实现。
4. kubernetes: 提供 ClientSet 客户端。
5. listers: 为每一个 K8S 资源提供 Lister 功能，该功能对 Get 和 List 请求提供只读的缓存数据。
6. plugin: 提供 OpenStack，GCP 和 Azure 等云服务商授权插件。
7. rest: 提供 RESTClient 客户端，对 K8S API Server 执行 RESTful 操作。
8. scale: 提供 ScaleClient 客户端，用于扩容或缩容 Deployment, Replicaset, Replication Controller 等资源对象。
9. tools: 提供常用工具，例如 SharedInformer, Reflector, DeltaFIFO 及 Indexers。 提供 Client 查询和缓存机制，以减少向 kube-apiserver 发起的请求数等。主要子目录为/tools/cache。
10. transport: 提供安全的 TCP 连接，支持 HTTP Stream，某些操作需要在客户端和容器之间传输二进制流，例如 exec，attach 等操作。该功能由内部的 SPDY 包提供支持。
11. util: 提供常用方法。例如 WorkQueue 工作队列，Certificate 证书管理等。

### client-go 组件
1. Reflector: 定义在 /tools/cache 包内的 Reflector 类型 中的 reflector 监视 Kubernetes API 以获取指定的资源类型 (Kind)。完成此操作的函数是 ListAndWatch。监视可以用于内建资源，也可以用于自定义资源。当 reflector 通过监视 API 的收到关于新资源实例存在的通知时，它使用相应的 listing API 获取新创建的对象，并将其放入 watchHandler 函数内的 Delta Fifo 队列中。
2. Informer: 在 /tools/cache 包内的基础 controller 中定义的一个 informer 从 Delta FIFO 队列中弹出对象。完成此操作的函数是 processLoop。这个基础 controller 的任务是保存对象以供以后检索，并调用 controller 将对象传递给它。
3. Indexer: indexer 为对象提供索引功能。它定义在 /tools/cache 包内的 Indexer 类型。一个典型的索引用例是基于对象标签创建索引。Indexer 可以基于多个索引函数维护索引。Indexer 使用线程安全的数据存储来存储对象及其键值。在 /tools/cache 包内的 Store 类型 定义了一个名为MetaNamespaceKeyFunc的默认函数，该函数为该对象生成一个名为 <namespace>/<name> 组合的对象键值。

### Custom Controller 组件
1. Informer reference: 这是一个知道如何使用自定义资源对象的 Informer 实例的引用。您的自定义控制器代码需要创建适当的 Informer。
2. Indexer reference: 这是一个知道如何使用自定义资源对象的 Indexer 实例的引用。您的自定义控制器代码需要创建这个。您将使用此引用检索对象，以便稍后处理。
3. Resource Event Handlers: 当 Informer 想要分发一个对象给你的控制器时，会调用这些回调函数。编写这些函数的典型模式是获取已分配对象的键值，并将该键值放入一个工作队列中进行进一步处理。
4. Work queue: 这是在控制器代码中创建的队列，用于将对象的分发与处理解耦。编写 Resource Event Handler 函数来提取所分发对象的键值并将其添加到工作队列中。
5. Process Item: 这是在代码中创建的处理 work queue 中的 items 的函数。可以有一个或多个其他函数来执行实际的处理。这些函数通常使用 Indexer 引用 或 Listing wrapper 来获取与键值对应的对象。


## 参考文章
- [https://cloudnative.to/blog/client-go-study/](https://cloudnative.to/blog/client-go-study/)